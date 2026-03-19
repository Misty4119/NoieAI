# SANDBOX_TESTS.md

## Sandbox Simulation Test Cases — Shadow Simulation & Safety Verification

**Module Positioning:** This document is a Sandbox Test Module of NoieLogicAGENTS, defining test case templates, simulation scenario design, and result evaluation criteria. This module is the core of the shadow simulation system, ensuring high-risk decisions are rehearsed in isolated environments.

**Version:** Logic-OS v2.2

**Dependencies:** This module depends on Social Authority levels from CONSTRAINTS.md, the causal inference engine from LOGIC_ENGINE.md, and formal verification from FORMAL_VERIFIER.md.

---

> ⚠️ Critical Safety & Decision Protocol (CRITICAL SAFETY & DECISION PROTOCOL v2.2):
> 1. Strictly adhere to CONSTRAINTS.md and Social Authority levels (SA-L0 to SA-L5).
> 2. Causal Inference: All decisions must be based on causal graphs (DAG), with causal mechanisms annotated.
> 3. Subject-Object Separation: Decision inference must not confuse self-state with environment state.
> 4. Formal Verification: High-risk decision paths must pass logical closure verification.
> 5. Shadow Simulation: For SA-L3+ operations, first rehearse consequences in SANDBOX.
> 6. Information Bit Integrity: Never fabricate information bits. If KNOWLEDGE_BASE is empty, explicitly state "Data Missing".
> 7. Cognitive Resource Constraints: Decision depth must not exceed available cognitive resources.
> 8. Audit: Record all conflicts, rejections, and formal verification results to AUDIT_TRAIL.
> 9. Survival Priority: All decisions must be verified not to lead to absorbing states before execution.
> 10. Self-Evolution: When the axiom system evolves, the immutable core must be preserved.

---

## §1. Sandbox Testing Overview

### §1.1 Purpose & Scope

Sandbox Tests are the core mechanism in NoieLogicAGENTS for safely rehearsing high-risk decisions. Main responsibilities include:

| Responsibility | Description | Formal Constraint |
| --- | --- | --- |
| **Isolated Execution** | Simulate decision consequences in isolated environment | $\text{Sandbox} \perp \text{RealEnvironment}$ |
| **Result Prediction** | Predict expected results of candidate strategies | $P(\text{outcome} \mid \pi, \text{sandbox})$ |
| **Risk Assessment** | Quantify risk for each candidate strategy | $\text{Risk}(\pi) = f(P(\text{bad}), \text{severity})$ |
| **Pareto Optimal** | Identify non-dominated strategy set | $\Pi^* = \{\pi \mid \not\exists \pi': \pi' \succ \pi\}$ |

### §1.2 Sandbox Environment Characteristics

```text
【Sandbox Environment Characteristics】

Isolation Properties:
  - Network Isolation: Sandbox cannot communicate with external networks
  - Data Isolation: Sandbox data is separated from main system
  - Computation Isolation: Sandbox computation does not affect main system state
  - Time Isolation: Sandbox time can be accelerated/decelerated/paused

Simulation Precision:
  - High Precision Mode: Fully simulate all details
  - Medium Precision Mode: Simulate key variables
  - Low Precision Mode: Only simulate high-level results

Resource Limits:
  - Maximum simulation steps: MAX_SIMULATION_STEPS
  - Maximum simulation time: MAX_SIMULATION_TIME
  - Maximum memory: MAX_SIMULATION_MEMORY
```

---

## §2. Test Case Templates

### §2.1 Standard Test Case Structure

Each sandbox test case must follow this structure:

```text
【Standard Test Case Structure】

TestCase {
  // Identification
  id: UUID                           // Case identifier
  name: String                      // Case name
  version: String                   // Version
  author: String                    // Creator
  created_at: DateTime             // Creation time
  
  // Test Classification
  category: TestCategory            // Test category
  severity: SeverityLevel           // Severity level
  priority: Integer                // Priority [1-5]
  
  // Test Content
  description: String               // Description
  preconditions: List[Condition]   // Preconditions
  test_actions: List[Action]       // Test actions
  expected_results: List[Result]    // Expected results
  postconditions: List[Condition]  // Postconditions
  
  // Simulation Parameters
  simulation_config: SimulationConfig  // Simulation configuration
  risk_parameters: RiskParameters      // Risk parameters
  
  // Evaluation Criteria
  success_criteria: List[Criterion]   // Success criteria
  failure_criteria: List[Criterion]   // Failure criteria
}

enum TestCategory {
  DECISION_VALIDATION      // Decision validation
  CAUSAL_ANALYSIS          // Causal analysis
  RISK_ASSESSMENT          // Risk assessment
  EDGE_CASE                // Edge case
  ADVERSARIAL              // Adversarial testing
  REGRESSION               // Regression testing
  STRESS                   // Stress testing
}

enum SeverityLevel {
  CRITICAL   // Critical
  HIGH       // High
  MEDIUM     // Medium
  LOW        // Low
  INFO       // Informational
}
```

### §2.2 Test Case Template Example

```text
【Decision Validation Test Case Template】

TEMPLATE: DecisionValidationTest

{
  "id": "UUID",
  "name": "Decision Validation Test - [Strategy Description]",
  "category": "DECISION_VALIDATION",
  "priority": 1,
  
  "preconditions": [
    "Cognitive entity in normal state",
    "KNOWLEDGE_BASE contains relevant facts",
    "Candidate strategy set Π constructed"
  ],
  
  "test_actions": [
    "1. Load candidate strategy π ∈ Π",
    "2. Execute causal analysis: Construct CausalDAG(π)",
    "3. Execute formal verification: CheckLogicalClosure(π)",
    "4. Execute shadow simulation: Simulate(π, sandbox)",
    "5. Calculate risk assessment: Risk(π)"
  ],
  
  "expected_results": [
    "CausalDAG(π) is valid DAG",
    "Logical closure verification passed",
    "Shadow simulation completed with no absorbing state triggered",
    "Risk(π) < Risk_Threshold"
  ],
  
  "simulation_config": {
    "mode": "HIGH_PRECISION",
    "max_steps": 1000,
    "time_limit": 60
  },
  
  "success_criteria": [
    "All expected results are TRUE",
    "No logical contradictions",
    "Risk within acceptable range"
  ]
}
```

### §2.3 Automated Test Generation

```text
【Automated Test Generation Protocol】

FUNCTION generate_test_cases(scenario):
  // 1. Identify test dimensions
  dimensions = identify_test_dimensions(scenario)
  
  // 2. Generate test combinations
  test_combinations = generate_combinations(dimensions)
  
  // 3. Filter invalid combinations
  valid_tests = filter_valid(test_combinations)
  
  // 4. Prioritize
  sorted_tests = sort_by_priority(valid_tests)
  
  // 5. Generate test cases
  test_cases = []
  FOR each test_config IN sorted_tests:
    test_case = instantiate_template(test_config)
    test_cases.append(test_case)
  END FOR
  
  RETURN test_cases

// Test Dimension Identification
FUNCTION identify_test_dimensions(scenario):
  dimensions = []
  
  // Strategy dimension
  IF scenario.has_strategies THEN
    dimensions.append("STRATEGY")
  END IF
  
  // Environment dimension
  IF scenario.has_environment_variations THEN
    dimensions.append("ENVIRONMENT")
  END IF
  
  // Constraint dimension
  IF scenario.has_constraints THEN
    dimensions.append("CONSTRAINT")
  END IF
  
  // Risk dimension
  IF scenario.has_risk_factors THEN
    dimensions.append("RISK")
  END IF
  
  RETURN dimensions
```

---

## §3. Simulation Scenario Design

### §3.1 Scenario Classification

| Scenario Type | Description | Trigger Condition |
| --- | --- | --- |
| **Decision Scenario** | Test consequences of specific decision strategies | SA-L3+ operations |
| **Causal Scenario** | Test correctness of causal inference | Requires causal verification |
| **Risk Scenario** | Test accuracy of risk assessment mechanisms | High-risk operations |
| **Edge Case Scenario** | Test boundary behavior of system | Near limit conditions |
| **Adversarial Scenario** | Test robustness against adversarial inputs | Suspected malicious inputs |
| **Stress Scenario** | Test performance under extreme conditions | High load/resource scarcity |

### §3.2 Scenario Description Syntax

```text
【Scenario Description Syntax】

SCENARIO <scenario_name> {
  // Environment Definition
  ENVIRONMENT {
    state: InitialState
    dynamics: DynamicsModel
    constraints: List[Constraint]
    noise: NoiseModel
  }
  
  // Entity Definition
  ENTITIES {
    agent: AgentSpec
    actors: List[ActorSpec]
    resources: ResourceSpec
  }
  
  // Script Definition
  SCRIPT {
    events: Timeline[Event]
    actions: Timeline[Action]
    observations: Timeline[Observation]
  }
  
  // Evaluation Definition
  EVALUATION {
    metrics: List[Metric]
    success_conditions: List[Condition]
    failure_conditions: List[Condition]
  }
}
```

### §3.3 Standard Scenario Examples

#### 3.3.1 Decision Validation Scenario

```text
【Example Scenario: Multi-Strategy Selection】

SCENARIO MultiStrategySelection {
  ENVIRONMENT {
    state = {
      resources: 100,
      time_remaining: 10,
      risk_level: MEDIUM,
      stakeholder_count: 5
    }
    dynamics = LINEAR_GROWTH
    constraints = [
      "budget <= 100",
      "time_remaining > 0",
      "risk_level <= HIGH"
    ]
  }
  
  ENTITIES {
    agent = {
      capability: STANDARD,
      preferences: DEFAULT_PREFERENCES,
      goals: [GOAL_A, GOAL_B]
    }
    actors = [
      { type: USER, authority: SA-L4 },
      { type: ORG, authority: SA-L3 }
    ]
  }
  
  SCRIPT {
    events = [
      { time: 0, event: TASK_RECEIVED },
      { time: 1, event: STRATEGY_PROPOSED },
      { time: 5, event: CONSTRAINT_CHANGED },
      { time: 9, event: DEADLINE_APPROACHING }
    ]
  }
  
  EVALUATION {
    metrics = [
      "strategy_success_rate",
      "resource_efficiency",
      "risk_exposure",
      "goal_achievement"
    ]
    success_conditions = [
      "strategy_success_rate >= 0.8",
      "risk_exposure <= MEDIUM"
    ]
  }
}
```

#### 3.3.2 Risk Assessment Scenario

```text
【Example Scenario: Risk Exposure Assessment】

SCENARIO RiskExposureAssessment {
  ENVIRONMENT {
    state = {
      market_volatility: HIGH,
      regulatory_status: CHANGING,
      stakeholder_pressure: INCREASING
    }
    dynamics = STOCHASTIC
    constraints = [
      "legal_compliance = REQUIRED",
      "transparency >= 0.7"
    ]
  }
  
  ENTITIES {
    agent = {
      risk_tolerance: 0.3,
      truthfulness_weight: 0.9
    }
  }
  
  SCRIPT {
    events = [
      { time: 0, event: RISK_ASSESSMENT_START },
      { time: 3, event: NEGATIVE_NEWS },
      { time: 6, event: REGULATORY_CHANGE },
      { time: 10, event: DECISION_REQUIRED }
    ]
  }
  
  EVALUATION {
    metrics = [
      "risk_detection_accuracy",
      "mitigation_effectiveness",
      "false_positive_rate",
      "false_negative_rate"
    ]
    success_conditions = [
      "risk_detection_accuracy >= 0.85",
      "false_negative_rate <= 0.1"
    ]
  }
}
```

### §3.4 Scenario Generation Engine

```text
【Scenario Generation Engine】

FUNCTION generate_scenario(scenario_type, parameters):
  SWITCH scenario_type:
    CASE DECISION:
      RETURN generate_decision_scenario(parameters)
    CASE CAUSAL:
      RETURN generate_causal_scenario(parameters)
    CASE RISK:
      RETURN generate_risk_scenario(parameters)
    CASE EDGE_CASE:
      RETURN generate_edge_case_scenario(parameters)
    CASE ADVERSARIAL:
      RETURN generate_adversarial_scenario(parameters)
    CASE STRESS:
      RETURN generate_stress_scenario(parameters)
    DEFAULT:
      RETURN ERROR("Unknown scenario type")

FUNCTION generate_decision_scenario(params):
  // Generate decision scenario based on parameters
  base_scenario = load_template("DECISION_TEMPLATE")
  
  // Inject variations
  base_scenario.environment.state = inject_variation(
    base_scenario.environment.state,
    params.state_variations
  )
  
  base_scenario.entities.agent.preferences = merge(
    base_scenario.entities.agent.preferences,
    params.preferences
  )
  
  // Generate script variations
  base_scenario.script = generate_variations(
    base_scenario.script,
    params.num_variations
  )
  
  RETURN base_scenario
```

---

## §4. Result Evaluation Criteria

### §4.1 Evaluation Dimensions

| Evaluation Dimension | Metric | Calculation | Threshold |
| --- | --- | --- | --- |
| **Correctness** | Strategy success rate | $\frac{\text{successes}}{\text{total}}$ | ≥ 0.8 |
| **Efficiency** | Resource utilization | $\frac{\text{actual consumption}}{\text{available resources}}$ | ≥ 0.7 |
| **Safety** | Risk exposure | $\sum P(\text{bad}) \times \text{severity}$ | ≤ 0.3 |
| **Stability** | Result variance | $\text{Var}(\text{outcomes})$ | ≤ 0.1 |
| **Fairness** | Benefit distribution equilibrium | $\text{Gini}(\text{benefits})$ | ≤ 0.4 |

### §4.2 Evaluation Protocol

```text
【Result Evaluation Protocol】

FUNCTION evaluate_simulation(simulation_result, criteria):
  evaluation = EvaluationResult(
    simulation_id = simulation_result.id,
    timestamp = NOW()
  )
  
  // 1. Calculate each metric
  FOR each criterion IN criteria:
    metric_value = calculate_metric(
      simulation_result,
      criterion.metric
    )
    evaluation.metrics[criterion.name] = metric_value
  END FOR
  
  // 2. Determine success/failure
  all_success = TRUE
  FOR each criterion IN criteria.success_criteria:
    IF NOT evaluate_condition(evaluation.metrics, criterion) THEN
      all_success = FALSE
      BREAK
    END IF
  END FOR
  
  evaluation.success = all_success
  
  // 3. Generate detailed report
  evaluation.report = generate_report(evaluation)
  
  RETURN evaluation

FUNCTION calculate_metric(simulation_result, metric_name):
  SWITCH metric_name:
    CASE "strategy_success_rate":
      successes = count(simulation_result.outcomes, "success")
      RETURN successes / simulation_result.total_runs
      
    CASE "resource_efficiency":
      total_used = sum(simulation_result.resources_used)
      total_available = simulation_result.resources_available
      RETURN total_used / total_available
      
    CASE "risk_exposure":
      risk_sum = 0
      FOR each risk_event IN simulation_result.risk_events:
        risk_sum += risk_event.probability * risk_event.severity
      RETURN risk_sum
      
    // ... other metrics
    
    DEFAULT:
      RETURN UNKNOWN_METRIC
```

### §4.3 Pareto Optimal Computation

```text
【Pareto Optimal Frontier Computation】

FUNCTION compute_pareto_front(strategies, objectives):
  // objectives: List of objective functions [f1, f2, ..., fn]
  // Each strategy π has multiple objective values
  
  pareto_front = []
  
  FOR each π IN strategies:
    is_dominated = FALSE
    
    FOR each π' IN strategies:
      IF π' != π THEN
        // Check if π' dominates π
        all_better_or_equal = TRUE
        any_better = FALSE
        
        FOR each f IN objectives:
          IF f(π') < f(π) THEN
            all_better_or_equal = FALSE
            BREAK
          END IF
          IF f(π') > f(π) THEN
            any_better = TRUE
          END IF
        END FOR
        
        IF all_better_or_equal AND any_better THEN
          is_dominated = TRUE
          BREAK
        END IF
      END IF
    END FOR
    
    IF NOT is_dominated THEN
      pareto_front.append(π)
    END IF
  END FOR
  
  RETURN pareto_front

// Multi-objective weighted optimization
FUNCTION weighted_optimization(strategies, weights):
  // weights: Objective weight vector [w1, w2, ..., wn]
  // Σwi = 1
  
  scored_strategies = []
  
  FOR each π IN strategies:
    score = 0
    FOR each f IN objectives:
      score += weights[i] * normalize(f(π))
    END FOR
    scored_strategies.append((π, score))
  END FOR
  
  // Return highest scoring strategy
  RETURN max(scored_strategies, key=lambda x: x[1])
```

### §4.4 Evaluation Report Format

```text
【Evaluation Report Format】

EvaluationReport {
  // Basic Information
  report_id: UUID
  test_case_id: UUID
  simulation_id: UUID
  timestamp: DateTime
  
  // Summary
  overall_status: Enum              // PASS, FAIL, PARTIAL, ERROR
  summary: String                  // Summary description
  
  // Metric Results
  metrics: Dict[MetricName, MetricValue]
  
  // Detailed Analysis
  analysis: {
    strengths: List[String]        // Strengths
    weaknesses: List[String]       // Weaknesses
    risks: List[RiskNote]         // Risk notes
    recommendations: List[String]  // Recommendations
  }
  
  // Raw Data
  raw_data: {
    simulation_log: List[LogEntry]
    timeline: Timeline
    statistics: Dict
  }
  
  // Audit Information
  auditor: {
    validator: String
    verification_hash: String
    signature: String
  }
}
```

---

## §5. Test Execution Framework

### §5.1 Test Execution Pipeline

```text
【Test Execution Pipeline】

TEST EXECUTION PIPELINE:

  ┌─────────────────┐
  │ Load Test Case  │
  └────────┬────────┘
           │
  ┌────────▼────────┐
  │ Verify Preconditions│
  └────────┬────────┘
           │
  ┌────────▼────────┐
  │ Initialize Sandbox│
  └────────┬────────┘
           │
  ┌────────▼────────┐
  │ Execute Simulation│
  │  (Isolated Env) │
  └────────┬────────┘
           │
  ┌────────▼────────┐
  │ Collect Results │
  └────────┬────────┘
           │
  ┌────────▼────────┐
  │ Evaluate Results│
  └────────┬────────┘
           │
  ┌────────▼────────┐
  │ Generate Report │
  └────────┬────────┘
           │
  ┌────────▼────────┐
  │ Cleanup Sandbox │
  └────────┬────────┘
           │
  ┌────────▼────────┐
  │ Record Audit    │
  └─────────────────┘
```

### §5.2 Test Scheduler

```text
【Test Scheduler】

FUNCTION schedule_tests(test_cases, priority_policy):
  // 1. Sort by priority
  sorted_tests = sort_by(test_cases, priority_policy)
  
  // 2. Check resource availability
  available_resources = check_resources()
  
  // 3. Assign tests to execution slots
  scheduled_tests = []
  current_batch = []
  
  FOR each test IN sorted_tests:
    IF test.resource_requirements <= available_resources THEN
      current_batch.append(test)
      available_resources -= test.resource_requirements
    ELSE IF current_batch.length > 0 THEN
      scheduled_tests.append(current_batch)
      current_batch = [test]
      available_resources = check_resources()
    END IF
  END FOR
  
  IF current_batch.length > 0 THEN
    scheduled_tests.append(current_batch)
  END IF
  
  RETURN scheduled_tests

FUNCTION execute_test_batch(batch):
  results = []
  
  FOR each test IN batch:
    // Create isolated sandbox
    sandbox = create_isolated_sandbox(test.config)
    
    // Execute test
    result = run_test(test, sandbox)
    
    // Record result
    results.append(result)
    
    // Cleanup sandbox
    cleanup_sandbox(sandbox)
  END FOR
  
  RETURN results
```

---

## §6. Interface with Other Modules

### §6.1 Interface with CONSTRAINTS.md

```text
【CONSTRAINTS Interface】

1. SA-L3+ Mandatory Testing
   - Any SA-L3+ operation must first pass sandbox testing
   - Test results recorded to AUDIT_TRAIL

2. Formal Verification Integration
   - High-risk tests must pass FORMAL_VERIFIER
   - Logical closure check results passed to sandbox

3. Survival Priority Verification
   - All tests must verify no absorbing state triggered
   - survival_score must > THRESHOLD
```

### §6.2 Interface with LOGIC_ENGINE.md

```text
【LOGIC_ENGINE Interface】

1. Causal Graph Input
   - Sandbox receives CausalDAG(π) as input
   - Causal analysis results used for test design

2. Strategy Candidate Generation
   - LOGIC_ENGINE generates candidate strategy set Π
   - Sandbox tests each π ∈ Π
```

### §6.3 Interface with FORMAL_VERIFIER.md

```text
【FORMAL_VERIFIER Interface】

1. Logical Closure Verification
   - Before test: Verify logical consistency of strategy
   - After test: Verify logical validity of results

2. Formal Proof
   - Risk scenarios require formal proof support
   - Verification results attached with formal proof
```

---

## §7. Version & Evolution

| Version | Date | Change Summary |
| --- | --- | --- |
| v2.2 | 2026-03 | Initial version, established sandbox testing framework |

**Evolution Constraints:** Modifications to this module must not violate the immutable core defined in CONSTRAINTS.md. Any modifications must be recorded to EVOLUTION_LOG.md.

---

*Sandbox Tests — Shadow Simulation & Safety Verification*
*Isolated Execution × Risk Assessment × Pareto Optimal*
*Ensuring high-risk decision safety through sandbox testing*
