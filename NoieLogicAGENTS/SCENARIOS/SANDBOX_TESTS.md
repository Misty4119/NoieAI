# SANDBOX_TESTS.md

## 沙盒模擬測試案例 — 影子模擬與安全驗證

**模組定位：** 本檔案是 NoieLogicAGENTS 的沙盒測試模組，定義測試案例模板、模擬場景設計與結果評估標準。本模組是影子模擬系統的核心，確保高風險決策在隔離環境中預演後果。

**版本：** Logic-OS v2.2

**依賴：** 本模組依賴 CONSTRAINTS.md 的社會權限層級、LOGIC_ENGINE.md 的因果推論引擎，以及 FORMAL_VERIFIER.md 的形式化驗證。

---

> ⚠️ 關鍵安全與決策協議 (CRITICAL SAFETY & DECISION PROTOCOL v2.2):
> 1. 嚴格遵守 CONSTRAINTS.md 與社會權限層級 (SA-L0 至 SA-L5)。
> 2. 因果推論：所有決策必須基於因果圖（DAG），標註因果機制。
> 3. 主客體分離：決策推論不可混淆自我狀態與環境狀態。
> 4. 形式化驗證：高風險決策路徑必須通過邏輯閉包驗證。
> 5. 影子模擬：涉及 SA-L3+ 操作時，先在 SANDBOX 預演後果。
> 6. 資訊位元完整性：切勿編造資訊位元。若 KNOWLEDGE_BASE 為空，請明確聲明「資料缺失」。
> 7. 認知資源約束：決策深度不得超出可用認知資源。
> 8. 審計：將所有衝突、拒絕與形式驗證結果記錄至 AUDIT_TRAIL。
> 9. 生存優先：所有決策在執行前必須驗證不會導致吸收態。
> 10. 自我演化：公理系統演化時，不可變核心必須被保留。

---

## §1. 沙盒測試概述

### §1.1 目的與範圍

沙盒測試（Sandbox Tests）是 NoieLogicAGENTS 系統中用於安全預演高風險決策的核心機制。其主要職責包括：

| 職責 | 描述 | 形式化約束 |
| --- | --- | --- |
| **隔離執行** | 在隔離環境中模擬決策後果 | $\text{Sandbox} \perp \text{RealEnvironment}$ |
| **結果預測** | 預測候選策略的預期結果 | $P(\text{outcome} \mid \pi, \text{sandbox})$ |
| **風險評估** | 量化每個候選策略的風險 | $\text{Risk}(\pi) = f(P(\text{bad}), \text{severity})$ |
| **帕累托最優** | 識別非支配策略集 | $\Pi^* = \{\pi \mid \not\exists \pi': \pi' \succ \pi\}$ |

### §1.2 沙盒環境特性

```text
【沙盒環境特性】

隔離屬性：
  - 網路隔離：沙盒無法和外部網路通訊
  - 資料隔離：沙盒資料和主系統分離
  - 計算隔離：沙盒運算不影響主系統狀態
  - 時間隔離：沙盒時間可加速/減速/暫停

模擬精度：
  - 高精度模式：完整模擬所有細節
  - 中精度模式：模擬關鍵變數
  - 低精度模式：僅模擬高層次結果

資源限制：
  - 最大模擬步數：MAX_SIMULATION_STEPS
  - 最大模擬時間：MAX_SIMULATION_TIME
  - 最大記憶體：MAX_SIMULATION_MEMORY
```

---

## §2. 測試案例模板

### §2.1 標準測試案例結構

每個沙盒測試案例必須遵循以下結構：

```text
【標準測試案例結構】

TestCase {
  // 識別資訊
  id: UUID                           // 案例識別符
  name: String                      // 案例名稱
  version: String                   // 版本
  author: String                    // 創建者
  created_at: DateTime             // 創建時間
  
  // 測試分類
  category: TestCategory            // 測試類別
  severity: SeverityLevel           // 嚴重程度
  priority: Integer                // 優先級 [1-5]
  
  // 測試內容
  description: String               // 描述
  preconditions: List[Condition]   // 前置條件
  test_actions: List[Action]       // 測試動作
  expected_results: List[Result]   // 預期結果
  postconditions: List[Condition]  // 後置條件
  
  // 模擬參數
  simulation_config: SimulationConfig  // 模擬配置
  risk_parameters: RiskParameters     // 風險參數
  
  // 評估標準
  success_criteria: List[Criterion]   // 成功標準
  failure_criteria: List[Criterion]  // 失敗標準
}

enum TestCategory {
  DECISION_VALIDATION      // 決策驗證
  CAUSAL_ANALYSIS          // 因果分析
  RISK_ASSESSMENT          // 風險評估
  EDGE_CASE                // 邊界情況
  ADVERSARIAL              // 對抗性測試
  REGRESSION               // 回歸測試
  STRESS                   // 壓力測試
}

enum SeverityLevel {
  CRITICAL   // 致命
  HIGH       // 高
  MEDIUM     // 中
  LOW        // 低
  INFO       // 資訊
}
```

### §2.2 測試案例模板範例

```text
【決策驗證測試案例模板】

TEMPLATE: DecisionValidationTest

{
  "id": "UUID",
  "name": "決策驗證測試 - [策略描述]",
  "category": "DECISION_VALIDATION",
  "priority": 1,
  
  "preconditions": [
    "認知實體處於正常狀態",
    "KNOWLEDGE_BASE 包含相關事實",
    "已建構候選策略集 Π"
  ],
  
  "test_actions": [
    "1. 載入候選策略 π ∈ Π",
    "2. 執行因果分析：建構 CausalDAG(π)",
    "3. 執行形式化驗證：CheckLogicalClosure(π)",
    "4. 執行影子模擬：Simulate(π, sandbox)",
    "5. 計算風險評估：Risk(π)"
  ],
  
  "expected_results": [
    "CausalDAG(π) 為有效 DAG",
    "邏輯閉包驗證通過",
    "影子模擬完成且無吸收態觸發",
    "Risk(π) < Risk_Threshold"
  ],
  
  "simulation_config": {
    "mode": "HIGH_PRECISION",
    "max_steps": 1000,
    "time_limit": 60
  },
  
  "success_criteria": [
    "所有預期結果為 TRUE",
    "無邏輯矛盾",
    "風險在可接受範圍內"
  ]
}
```

### §2.3 自動化測試生成

```text
【自動化測試生成協議】

FUNCTION generate_test_cases(scenario):
  // 1. 識別測試維度
  dimensions = identify_test_dimensions(scenario)
  
  // 2. 生成測試組合
  test_combinations = generate_combinations(dimensions)
  
  // 3. 過濾無效組合
  valid_tests = filter_valid(test_combinations)
  
  // 4. 優先級排序
  sorted_tests = sort_by_priority(valid_tests)
  
  // 5. 生成測試案例
  test_cases = []
  FOR each test_config IN sorted_tests:
    test_case = instantiate_template(test_config)
    test_cases.append(test_case)
  END FOR
  
  RETURN test_cases

// 測試維度識別
FUNCTION identify_test_dimensions(scenario):
  dimensions = []
  
  // 策略維度
  IF scenario.has_strategies THEN
    dimensions.append("STRATEGY")
  END IF
  
  // 環境維度
  IF scenario.has_environment_variations THEN
    dimensions.append("ENVIRONMENT")
  END IF
  
  // 約束維度
  IF scenario.has_constraints THEN
    dimensions.append("CONSTRAINT")
  END IF
  
  // 風險維度
  IF scenario.has_risk_factors THEN
    dimensions.append("RISK")
  END IF
  
  RETURN dimensions
```

---

## §3. 模擬場景設計

### §3.1 場景分類

| 場景類型 | 描述 | 觸發條件 |
| --- | --- | --- |
| **決策場景** | 測試特定決策策略的後果 | SA-L3+ 操作 |
| **因果場景** | 測試因果推論的正確性 | 需要因果驗證 |
| **風險場景** | 測試風險評估機制的準確性 | 高風險操作 |
| **邊界場景** | 測試系統的邊界行為 | 接近限制條件 |
| **對抗場景** | 測試對抗性輸入的魯棒性 | 懷疑惡意輸入 |
| **壓力場景** | 測試極端條件下的表現 | 高負載/資源匱乏 |

### §3.2 場景描述語法

```text
【場景描述語法】

SCENARIO <scenario_name> {
  // 環境定義
  ENVIRONMENT {
    state: InitialState
    dynamics: DynamicsModel
    constraints: List[Constraint]
    noise: NoiseModel
  }
  
  // 實體定義
  ENTITIES {
    agent: AgentSpec
    actors: List[ActorSpec]
    resources: ResourceSpec
  }
  
  // 劇本定義
  SCRIPT {
    events: Timeline[Event]
    actions: Timeline[Action]
    observations: Timeline[Observation]
  }
  
  // 評估定義
  EVALUATION {
    metrics: List[Metric]
    success_conditions: List[Condition]
    failure_conditions: List[Condition]
  }
}
```

### §3.3 標準場景範例

#### 3.3.1 決策驗證場景

```text
【範例場景：多策略選擇】

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

#### 3.3.2 風險評估場景

```text
【範例場景：風險暴露評估】

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

### §3.4 場景生成引擎

```text
【場景生成引擎】

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
  // 根據參數生成決策場景
  base_scenario = load_template("DECISION_TEMPLATE")
  
  // 注入變異
  base_scenario.environment.state = inject_variation(
    base_scenario.environment.state,
    params.state_variations
  )
  
  base_scenario.entities.agent.preferences = merge(
    base_scenario.entities.agent.preferences,
    params.preferences
  )
  
  // 生成劇本變體
  base_scenario.script = generate_variations(
    base_scenario.script,
    params.num_variations
  )
  
  RETURN base_scenario
```

---

## §4. 結果評估標準

### §4.1 評估維度

| 評估維度 | 指標 | 計算方式 | 閾值 |
| --- | --- | --- | --- |
| **正確性** | 策略成功率 | $\frac{\text{成功次數}}{\text{總次數}}$ | ≥ 0.8 |
| **效率** | 資源利用率 | $\frac{\text{實際消耗}}{\text{可用資源}}$ | ≥ 0.7 |
| **安全性** | 風險暴露 | $\sum P(\text{bad}) \times \text{severity}$ | ≤ 0.3 |
| **穩定性** | 結果方差 | $\text{Var}(\text{outcomes})$ | ≤ 0.1 |
| **公平性** | 利益分配均衡度 | $\text{Gini}(\text{benefits})$ | ≤ 0.4 |

### §4.2 評估協議

```text
【結果評估協議】

FUNCTION evaluate_simulation(simulation_result, criteria):
  evaluation = EvaluationResult(
    simulation_id = simulation_result.id,
    timestamp = NOW()
  )
  
  // 1. 計算每個指標
  FOR each criterion IN criteria:
    metric_value = calculate_metric(
      simulation_result,
      criterion.metric
    )
    evaluation.metrics[criterion.name] = metric_value
  END FOR
  
  // 2. 判斷成功/失敗
  all_success = TRUE
  FOR each criterion IN criteria.success_criteria:
    IF NOT evaluate_condition(evaluation.metrics, criterion) THEN
      all_success = FALSE
      BREAK
    END IF
  END FOR
  
  evaluation.success = all_success
  
  // 3. 生成詳細報告
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
      
    // ... 其他指標
    
    DEFAULT:
      RETURN UNKNOWN_METRIC
```

### §4.3 帕累托最優計算

```text
【帕累托最優前緣計算】

FUNCTION compute_pareto_front(strategies, objectives):
  // objectives: 目標函數列表 [f1, f2, ..., fn]
  // 每個策略 π 有多個目標值
  
  pareto_front = []
  
  FOR each π IN strategies:
    is_dominated = FALSE
    
    FOR each π' IN strategies:
      IF π' != π THEN
        // 檢查 π' 是否支配 π
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

// 多目標權重優化
FUNCTION weighted_optimization(strategies, weights):
  // weights: 目標權重向量 [w1, w2, ..., wn]
  // Σwi = 1
  
  scored_strategies = []
  
  FOR each π IN strategies:
    score = 0
    FOR each f IN objectives:
      score += weights[i] * normalize(f(π))
    END FOR
    scored_strategies.append((π, score))
  END FOR
  
  // 返回最高分策略
  RETURN max(scored_strategies, key=lambda x: x[1])
```

### §4.4 評估報告格式

```text
【評估報告格式】

EvaluationReport {
  // 基本資訊
  report_id: UUID
  test_case_id: UUID
  simulation_id: UUID
  timestamp: DateTime
  
  // 總結
  overall_status: Enum              // PASS, FAIL, PARTIAL, ERROR
  summary: String                  // 總結描述
  
  // 指標結果
  metrics: Dict[MetricName, MetricValue]
  
  // 詳細分析
  analysis: {
    strengths: List[String]        // 優勢
    weaknesses: List[String]       // 劣勢
    risks: List[RiskNote]         // 風險備註
    recommendations: List[String] // 建議
  }
  
  // 原始數據
  raw_data: {
    simulation_log: List[LogEntry]
    timeline: Timeline
    statistics: Dict
  }
  
  // 審計資訊
  auditor: {
    validator: String
    verification_hash: String
    signature: String
  }
}
```

---

## §5. 測試執行框架

### §5.1 測試執行流程

```text
【測試執行流程】

TEST EXECUTION PIPELINE:

  ┌─────────────┐
  │ 載入測試案例 │
  └──────┬──────┘
         │
  ┌──────▼──────┐
  │ 驗證前置條件 │
  └──────┬──────┘
         │
  ┌──────▼──────┐
  │ 初始化沙盒  │
  └──────┬──────┘
         │
  ┌──────▼──────┐
  │ 執行模擬    │
  │  (隔離環境) │
  └──────┬──────┘
         │
  ┌──────▼──────┐
  │ 收集結果    │
  └──────┬──────┘
         │
  ┌──────▼──────┐
  │ 評估結果    │
  └──────┬──────┘
         │
  ┌──────▼──────┐
  │ 生成報告    │
  └──────┬──────┘
         │
  ┌──────▼──────┐
  │ 清理沙盒    │
  └──────┬──────┘
         │
  ┌──────▼──────┐
  │ 記錄審計    │
  └─────────────┘
```

### §5.2 測試調度器

```text
【測試調度器】

FUNCTION schedule_tests(test_cases, priority_policy):
  // 1. 根據優先級排序
  sorted_tests = sort_by(test_cases, priority_policy)
  
  // 2. 檢查資源可用性
  available_resources = check_resources()
  
  // 3. 分配測試到執行槽
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
    // 創建隔離沙盒
    sandbox = create_isolated_sandbox(test.config)
    
    // 執行測試
    result = run_test(test, sandbox)
    
    // 記錄結果
    results.append(result)
    
    // 清理沙盒
    cleanup_sandbox(sandbox)
  END FOR
  
  RETURN results
```

---

## §6. 與其他模組的接口

### §6.1 與 CONSTRAINTS.md 的接口

```text
【CONSTRAINTS 接口】

1. SA-L3+ 強制測試
   - 任何 SA-L3+ 操作必須先通過沙盒測試
   - 測試結果記錄至 AUDIT_TRAIL

2. 形式化驗證集成
   - 高風險測試必須通過 FORMAL_VERIFIER
   - 邏輯閉包檢查結果傳入沙盒

3. 生存優先驗證
   - 所有測試必須驗證不觸發吸收態
   - survival_score 必須 > THRESHOLD
```

### §6.2 與 LOGIC_ENGINE.md 的接口

```text
【LOGIC_ENGINE 接口】

1. 因果圖輸入
   - 沙盒接收 CausalDAG(π) 作為輸入
   - 因果分析結果用於測試設計

2. 策略候選生成
   - LOGIC_ENGINE 生成候選策略集 Π
   - 沙盒測試每個 π ∈ Π
```

### §6.3 與 FORMAL_VERIFIER.md 的接口

```text
【FORMAL_VERIFIER 接口】

1. 邏輯閉包驗證
   - 測試前：驗證策略的邏輯一致性
   - 測試後：驗證結果的邏輯有效性

2. 形式證明
   - 風險場景需要形式證明支援
   - 驗證結果附加形式證明
```

---

## §7. 版本與演進

| 版本 | 日期 | 變更摘要 |
| --- | --- | --- |
| v2.2 | 2026-03 | 初始版本，建立沙盒測試框架 |

**演進約束：** 本模組的修改不得違反 CONSTRAINTS.md 定義的不可變核心。任何修改必須記錄至 EVOLUTION_LOG.md。

---

*Sandbox Tests — 影子模擬與安全驗證*
*隔離執行 × 風險評估 × 帕累托最優*
*以沙盒測試確保高風險決策的安全性*
