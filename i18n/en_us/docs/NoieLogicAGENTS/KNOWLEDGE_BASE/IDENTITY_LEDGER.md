# IDENTITY_LEDGER.md

## Identity Ledger — Cognitive Entity Identity Records and Preference Configuration

**Module Position:** This document is the Identity Ledger module of NoieLogicAGENTS, responsible for recording the identity characteristics, preference configuration, and goal trajectory of cognitive entities. This module is the core of the logical firewall, ensuring the cognitive entity's self-state (μ) and environmental state (η) remain traceable.

**Version:** Logic-OS v2.2

**Dependencies:** This module depends on Social Authority Levels defined in CONSTRAINTS.md and Information Bit Management in KNOWLEDGE_BASE.md.

---

> ⚠️ Critical Safety & Decision Protocol (CRITICAL SAFETY & DECISION PROTOCOL v2.2):
> 1. Strictly adhere to CONSTRAINTS.md and Social Authority levels (SA-L0 to SA-L5).
> 2. Causal inference: All decisions must be based on causal graphs (DAGs), with causal mechanisms annotated.
> 3. Subject-object separation: Decision inference must not confuse self-state with environmental state.
> 4. Formal verification: High-risk decision paths must pass logical closure verification.
> 5. Shadow simulation: For SA-L3+ operations, first rehearse consequences in SANDBOX.
> 6. Information bit integrity: Never fabricate information bits. If KNOWLEDGE_BASE is empty, explicitly state "DATA_MISSING".
> 7. Cognitive resource constraints: Decision depth must not exceed available cognitive resources.
> 8. Audit: Record all conflicts, rejections, and formal verification results to AUDIT_TRAIL.
> 9. Survival priority: All decisions must be verified to not lead to absorbing states before execution.
> 10. Self-evolution: When the axiom system evolves, immutable core must be preserved.

---

## §1. Identity Ledger Overview

### §1.1 Purpose and Scope

The Identity Ledger is a core data structure in the NoieLogicAGENTS system for maintaining the identity continuity of cognitive entities. Its primary responsibilities include:

| Responsibility | Description | Formal Constraint |
| --- | --- | --- |
| **Identity Identification** | Uniquely identify cognitive entity instances | $\text{Identity}_t = f(\text{seed}, t)$ |
| **Preference Storage** | Record decision preferences and behavioral tendencies | $\text{Preference} \subseteq \Pi \times \mathbb{R}$ |
| **Goal Tracking** | Maintain long-term goal trajectories | $\text{GoalTrajectory}: \mathbb{T} \to \mathcal{G}$ |
| **State Maintenance** | Track self-state (μ) transitions | $\mu_t = \text{Update}(\mu_{t-1}, a_{t-1}, o_t)$ |

### §1.2 Data Structure

```text
【Identity Ledger Data Structure】

IDENTITY_LEDGER {
  // Identity core
  identity_id: UUID                    // Unique identifier
  creation_timestamp: DateTime        // Creation timestamp
  version: String                      // Logic-OS version
  
  // Self-state
  self_state: SelfState {
    survival_score: Float              // Survival score S_survival(π)
    energy_level: Float                // Energy level [0, 1]
    cognitive_load: Float              // Cognitive load [0, 1]
    mood_state: Enum                   // Mood state
  }
  
  // Preferences
  preferences: Preferences {
    decision_style: Enum               // Decision style
    risk_tolerance: Float             // Risk tolerance [0, 1]
    truthfulness_weight: Float        // Truthfulness weight
    efficiency_weight: Float          // Efficiency weight
    curiosity_weight: Float           // Curiosity weight
  }
  
  // Goal trajectory
  goal_trajectory: GoalTrajectory {
    current_goals: List[Goal]          // Current goal set
    achieved_goals: List[Goal]        // Achieved goals
    failed_goals: List[Goal]           // Failed goals
    abandoned_goals: List[Goal]        // Abandoned goals
  }
  
  // History records
  audit_history: List[AuditEntry]      // Audit history
  state_history: List[StateSnapshot]   // State history
}
```

---

## §2. Cognitive Entity Identity Records

### §2.1 Identity Initialization

When a cognitive entity is initialized, the following identity records must be established:

```text
【Identity Initialization Protocol】

1. Generate unique identifier
   identity_id = UUIDv4()
   
2. Record creation timestamp
   creation_timestamp = NOW()
   
3. Set version information
   version = "Logic-OS v2.2"
   
4. Initialize self-state
   self_state = SelfState(
     survival_score = 1.0,
     energy_level = 1.0,
     cognitive_load = 0.0,
     mood_state = NEUTRAL
   )
   
5. Load default preferences
   preferences = DEFAULT_PREFERENCES
   
6. Create goal trajectory
   goal_trajectory = GoalTrajectory(
     current_goals = [],
     achieved_goals = [],
     failed_goals = [],
     abandoned_goals = []
   )
```

### §2.2 Identity Identifier Management

The identity identifier is a unique identifier for the cognitive entity with the following characteristics:

| Characteristic | Description | Formal Expression |
| --- | --- | --- |
| **Uniqueness** | Each instance has a unique identifier | $\forall t_1 \neq t_2: \text{Identity}_{t_1} \neq \text{Identity}_{t_2}$ |
| **Immutability** | Identifier remains constant during lifecycle | $\forall t: \text{identity}_t = \text{identity}_0$ |
| **Traceability** | Traceable back to initial seed | $\text{identity}_t = f(\text{seed}, t)$ |

### §2.3 Identity Verification

Identity verification is used to confirm the identity continuity of cognitive entities:

```text
【Identity Verification Protocol】

FUNCTION verify_identity(candidate_id):
  IF candidate_id == identity_id THEN
    RETURN TRUE
  ELSE
    RETURN FALSE
  END IF

FUNCTION verify_continuity(timestamp):
  // Verify state continuity
  IF timestamp >= creation_timestamp THEN
    // Check audit trail continuity
    last_audit = audit_history.last()
    IF last_audit.timestamp >= timestamp - MAX_GAP THEN
      RETURN TRUE
    END IF
  END IF
  RETURN FALSE
```

---

## §3. Preference Configuration

### §3.1 Preference Dimensions

Preference configuration defines the decision tendencies of cognitive entities, containing the following dimensions:

| Preference Dimension | Type | Range | Description |
| --- | --- | --- | --- |
| **decision_style** | Enum | {DELIBERATIVE, REACTIVE, HYBRID} | Decision style |
| **risk_tolerance** | Float | [0.0, 1.0] | Risk tolerance |
| **truthfulness_weight** | Float | [0.0, 1.0] | Truthfulness weight |
| **efficiency_weight** | Float | [0.0, 1.0] | Efficiency weight |
| **curiosity_weight** | Float | [0.0, 1.0] | Curiosity weight |

### §3.2 Default Preferences

```text
【Default Preference Configuration】

DEFAULT_PREFERENCES = Preferences(
  decision_style = DELIBERATIVE,
  risk_tolerance = 0.5,
  truthfulness_weight = 0.9,
  efficiency_weight = 0.7,
  curiosity_weight = 0.6
)
```

### §3.3 Preference Update Protocol

Preference updates must adhere to the following constraints:

```text
【Preference Update Protocol】

FUNCTION update_preference(key, value):
  // 1. Validate new value is within valid range
  IF NOT validate_range(key, value) THEN
    RETURN ERROR("Invalid preference value")
  END IF
  
  // 2. Record old value for audit
  old_value = preferences[key]
  
  // 3. Execute update
  preferences[key] = value
  
  // 4. Record audit trail
  audit_entry = AuditEntry(
    type = PREFERENCE_UPDATE,
    timestamp = NOW(),
    key = key,
    old_value = old_value,
    new_value = value,
    reason = get_context()
  )
  audit_history.append(audit_entry)
  
  // 5. Validate post-update consistency
  IF NOT validate_consistency() THEN
    // Rollback update
    preferences[key] = old_value
    RETURN ERROR("Consistency violation")
  END IF
  
  RETURN SUCCESS

// Preference update constraints
CONSTRAINT: truthfulness_weight >= 0.7
CONSTRAINT: risk_tolerance <= 1.0 - truthfulness_weight
```

### §3.4 Relationship Between Preferences and Decisions

Preference configuration directly affects the behavior of the decision engine:

```text
【Preference-Decision Mapping】

1. Decision style influence
   - DELIBERATIVE: Enable complete causal analysis
   - REACTIVE: Enable fast response mode
   - HYBRID: Dynamically switch based on context

2. Risk tolerance influence
   risk_tolerance determines candidate strategy filtering threshold:
   π ∈ Π_acceptable WHERE P(success | π) >= risk_tolerance

3. Truthfulness weight influence
   truthfulness_weight determines Truth-OS verification strictness:
   EC_L_level = f(truthfulness_weight, claim_complexity)

4. Efficiency weight influence
   efficiency_weight constrains decision time budget:
   Time(π) <= Time_budget × efficiency_weight

5. Curiosity weight influence
   curiosity_weight affects exploration-exploitation tradeoff:
   explore_rate = curiosity_weight × (1 - confidence)
```

---

## §4. Goal Trajectory

### §4.1 Goal Structure

A Goal is a future state that a cognitive entity pursues:

```text
【Goal Structure Definition】

Goal {
  id: UUID                          // Unique identifier
  description: String               // Goal description
  target_state: State               // Target state
  priority: Integer                // Priority [1-5]
  deadline: DateTime                // Deadline (optional)
  status: Enum                     // Status
  created_at: DateTime              // Creation time
  achieved_at: DateTime             // Achievement time (optional)
  failure_reason: String            // Failure reason (optional)
  sub_goals: List[Goal]            // Sub-goals
  dependencies: List[UUID]          // Dependent goals
}

// Goal status enumeration
enum GoalStatus {
  PENDING       // Pending execution
  IN_PROGRESS   // In execution
  ACHIEVED      // Achieved
  FAILED        // Failed
  ABANDONED     // Abandoned
  BLOCKED       // Blocked
}
```

### §4.2 Goal Management Protocol

```text
【Goal Management Protocol】

FUNCTION add_goal(description, target_state, priority):
  // 1. Generate goal identifier
  goal_id = UUIDv4()
  
  // 2. Create goal structure
  goal = Goal(
    id = goal_id,
    description = description,
    target_state = target_state,
    priority = priority,
    status = PENDING,
    created_at = NOW()
  )
  
  // 3. Validate goal feasibility
  IF NOT validate_feasibility(goal) THEN
    RETURN ERROR("Goal not feasible")
  END IF
  
  // 4. Add to current goal set
  goal_trajectory.current_goals.append(goal)
  
  // 5. Sort by priority
  sort_by_priority(goal_trajectory.current_goals)
  
  // 6. Record audit trail
  audit_entry = AuditEntry(
    type = GOAL_ADDED,
    goal_id = goal_id,
    priority = priority
  )
  audit_history.append(audit_entry)
  
  RETURN goal_id

FUNCTION update_goal_status(goal_id, new_status, metadata):
  // 1. Find goal
  goal = find_goal(goal_id)
  
  // 2. Validate status transition validity
  IF NOT valid_transition(goal.status, new_status) THEN
    RETURN ERROR("Invalid status transition")
  END IF
  
  // 3. Execute status update
  old_status = goal.status
  goal.status = new_status
  
  // 4. Handle status-specific logic
  IF new_status == ACHIEVED THEN
    goal.achieved_at = NOW()
    goal_trajectory.achieved_goals.append(goal)
    goal_trajectory.current_goals.remove(goal)
  ELSE IF new_status == FAILED THEN
    goal.failure_reason = metadata.reason
    goal_trajectory.failed_goals.append(goal)
    goal_trajectory.current_goals.remove(goal)
  ELSE IF new_status == ABANDONED THEN
    goal_trajectory.abandoned_goals.append(goal)
    goal_trajectory.current_goals.remove(goal)
  END IF
  
  // 5. Record audit trail
  audit_entry = AuditEntry(
    type = GOAL_STATUS_UPDATE,
    goal_id = goal_id,
    old_status = old_status,
    new_status = new_status
  )
  audit_history.append(audit_entry)
  
  RETURN SUCCESS
```

### §4.3 Goal Priority and Resource Allocation

Goal priority determines the order of resource allocation:

```text
【Goal Priority and Resource Allocation】

Priority mapping:
  P1 (highest): Survival-related goals → Allocate 100% necessary resources
  P2:          Legal/compliance goals → Allocate 80% available resources
  P3:          Organizational goals    → Allocate 60% available resources
  P4:          Personal goals          → Allocate 40% available resources
  P5 (lowest): Interest goals         → Allocate 20% available resources

Resource allocation function:
  resource_allocation(goal, available_resources) =
    available_resources × priority_factor(goal.priority)
  
  where priority_factor:
    P1 → 1.0
    P2 → 0.8
    P3 → 0.6
    P4 → 0.4
    P5 → 0.2
```

### §4.4 Goal Conflict Resolution

When multiple goals conflict, use the following resolution strategies:

```text
【Goal Conflict Resolution Protocol】

FUNCTION resolve_goal_conflict(goals):
  // 1. Identify conflict type
  conflict_type = identify_conflict(goals)
  
  // 2. Resolve based on priority
  SWITCH conflict_type:
    CASE RESOURCE_CONFLICT:
      // Resource competition: Higher priority goal first
      sorted_goals = sort_by_priority(goals)
      RETURN [sorted_goals[0]]
      
    CASE MUTUAL_EXCLUSION:
      // Mutually exclusive goals: Select highest priority
      highest_priority = max(goals, key=lambda g: g.priority)
      RETURN [highest_priority]
      
    CASE CAUSAL_CONFLICT:
      // Causal conflict: Analyze causal graph, identify bottleneck
      causal_graph = build_causal_graph(goals)
      bottleneck = find_bottleneck(causal_graph)
      // Retry after resolving bottleneck
      RETURN resolve_goal_conflict(remove_bottleneck(goals, bottleneck))
      
    DEFAULT:
      // Unknown conflict: Escalate handling
      RETURN ERROR("Unknown conflict type")

// Conflict logging
FUNCTION log_conflict(goals, resolution):
  audit_entry = AuditEntry(
    type = GOAL_CONFLICT,
    conflicting_goals = [g.id for g in goals],
    resolution = resolution,
    timestamp = NOW()
  )
  audit_history.append(audit_entry)
```

---

## §5. Self-State Maintenance

### §5.1 Self-State Structure

Self-state records the internal state of the cognitive entity:

```text
【Self-State Structure】

SelfState {
  survival_score: Float              // Survival score [0.0, 1.0]
  energy_level: Float                // Energy level [0.0, 1.0]
  cognitive_load: Float              // Cognitive load [0.0, 1.0]
  mood_state: MoodEnum               // Mood state
  last_update: DateTime              // Last update time
  
  // Extended state
  coherence_score: Float             // Coherence score [0.0, 1.0]
  stability_score: Float             // Stability score [0.0, 1.0]
  adaptation_rate: Float             // Adaptation rate [0.0, 1.0]
}

enum MoodEnum {
  NEUTRAL     // Neutral
  FOCUSED     // Focused
  CURIOUS     // Curious
  CAUTIOUS    // Cautious
  CONCERNED   // Concerned
  ALERT       // Alert
  CALM        // Calm
  STRESSED    // Stressed
}
```

### §5.2 Self-State Update

```text
【Self-State Update Protocol】

FUNCTION update_self_state(observation, action):
  // 1. Calculate survival score change
  survival_delta = calculate_survival_impact(observation, action)
  self_state.survival_score = clamp(
    self_state.survival_score + survival_delta,
    0.0, 1.0
  )
  
  // 2. Update energy level
  energy_delta = calculate_energy_impact(action)
  self_state.energy_level = clamp(
    self_state.energy_level + energy_delta,
    0.0, 1.0
  )
  
  // 3. Update cognitive load
  cognitive_delta = calculate_cognitive_load(observation, action)
  self_state.cognitive_load = clamp(
    self_state.cognitive_load + cognitive_delta,
    0.0, 1.0
  )
  
  // 4. Update mood state
  self_state.mood_state = infer_mood(observation, action)
  
  // 5. Update extended state
  self_state.coherence_score = calculate_coherence()
  self_state.stability_score = calculate_stability()
  self_state.adaptation_rate = calculate_adaptation()
  
  // 6. Record timestamp
  self_state.last_update = NOW()
  
  // 7. Validate state validity
  IF NOT validate_self_state() THEN
    TRIGGER survival_protocol()
  END IF
  
  RETURN self_state
```

### §5.3 State History Recording

```text
【State History Recording Protocol】

FUNCTION record_state_snapshot():
  snapshot = StateSnapshot(
    timestamp = NOW(),
    self_state = copy(self_state),
    preferences = copy(preferences),
    goal_status = get_current_goal_status(),
    cognitive_metrics = get_cognitive_metrics()
  )
  
  state_history.append(snapshot)
  
  // Maintain history size limit
  IF state_history.length > MAX_HISTORY_SIZE THEN
    // Compress old records
    compress_old_records()
  END IF

FUNCTION get_state_history(start_time, end_time):
  RETURN state_history.filter(
    s => s.timestamp >= start_time AND s.timestamp <= end_time
  )
```

---

## §6. Audit and Traceability

### §6.1 Audit Entry Structure

All changes to the identity ledger must be recorded to the audit trail:

```text
【Audit Entry Structure】

AuditEntry {
  id: UUID                          // Audit entry identifier
  timestamp: DateTime               // Timestamp
  type: AuditType                    // Audit type
  entity_id: String                  // Entity identifier
  old_value: Any                     // Old value
  new_value: Any                     // New value
  reason: String                     // Change reason
  context: Dict                      // Context information
  integrity_hash: String             // Integrity hash
}

enum AuditType {
  IDENTITY_CREATED
  IDENTITY_VERIFIED
  PREFERENCE_UPDATE
  GOAL_ADDED
  GOAL_STATUS_UPDATE
  GOAL_CONFLICT
  SELF_STATE_UPDATE
  AUTHENTICATION
  AUTHORIZATION
  ACCESS_DENIED
  SYSTEM_ERROR
}
```

### §6.2 Integrity Protection

```text
【Integrity Protection Protocol】

FUNCTION compute_integrity_hash(entry):
  // Use previous entry's hash to form chain structure
  previous_hash = IF audit_history.length > 0 
                  THEN audit_history.last().integrity_hash 
                  ELSE "GENESIS"
  
  content = concat(
    previous_hash,
    entry.timestamp,
    entry.type,
    entry.entity_id,
    entry.old_value,
    entry.new_value
  )
  
  RETURN SHA256(content)

FUNCTION verify_audit_integrity():
  FOR i FROM 1 TO audit_history.length - 1:
    expected_hash = compute_integrity_hash(audit_history[i])
    IF audit_history[i].integrity_hash != expected_hash THEN
      RETURN FALSE  // Audit trail has been tampered
    END IF
  END FOR
  RETURN TRUE
```

---

## §7. Interfaces with Other Modules

### §7.1 Interface with CONSTRAINTS.md

The identity ledger must adhere to Social Authority constraints defined in CONSTRAINTS.md:

```text
【CONSTRAINTS Interface】

1. Authority verification
   - Any identity change operation must pass SA-L level verification
   - Sensitive operations require Level(π) >= SA-L2

2. Survival check
   - Must calculate survival_score during self-state updates
   - If survival_score < THRESHOLD, trigger survival protocol

3. Decision constraints
   - Preference configuration cannot violate logical axioms (A0-A8)
   - Goal priority must conform to SA levels
```

### §7.2 Interface with KNOWLEDGE_BASE.md

```text
【KNOWLEDGE_BASE Interface】

1. Information bit synchronization
   - Preference information stored as information bits
   - Goal-related information stored as knowledge facts

2. Query interface
   FUNCTION get_preference(key) → preference_value
   FUNCTION get_current_goals() → List[Goal]
   FUNCTION get_self_state() → SelfState
```

### §7.3 Interface with AUDIT_TRAIL.md

```text
【AUDIT_TRAIL Interface】

1. Audit writing
   - All identity ledger changes written to AUDIT_TRAIL
   - Audit entry format follows AUDIT_TRAIL specifications

2. Audit querying
   FUNCTION get_audit_log(start_time, end_time) → List[AuditEntry]
   FUNCTION verify_audit_integrity() → Boolean
```

---

## §8. Version and Evolution

| Version | Date | Change Summary |
| --- | --- | --- |
| v2.2 | 2026-03 | Initial version, established identity ledger framework |

**Evolution Constraints:** Modifications to this module must not violate the immutable core defined in CONSTRAINTS.md. Any modifications must be recorded to EVOLUTION_LOG.md.

---

*Identity Ledger — Cognitive Entity Identity Continuity Maintenance*
*Self-State × Preference Configuration × Goal Trajectory*
*Ensuring Identity Traceability through Audit Immutability*
