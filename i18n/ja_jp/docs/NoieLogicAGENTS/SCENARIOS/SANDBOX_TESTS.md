# SANDBOX_TESTS.md

## サンドボックス模擬テストケース — 影子シミュレーションと安全検証

**モジュール位置：** 本ファイルは NoieLogicAGENTS のサンドボックステストモジュールであり、テストケーステンプレート、シミュレーションシナリオ設計、結果評価基準を定義します。本モジュールは影子シミュレーションシステムのコアであり、高リスク意思決定が隔離環境で結果を予備検討することを確保します。

**バージョン：** Logic-OS v2.2

**依存：** 本モジュールは CONSTRAINTS.md の社会権限レイヤー、LOGIC_ENGINE.md の因果推論エンジン、FORMAL_VERIFIER.md の形式的検証に依存します。

---

> ⚠️ 重要安全・意思決定プロトコル (CRITICAL SAFETY & DECISION PROTOCOL v2.2):
> 1. CONSTRAINTS.md および社会権限レイヤー（SA-L0 〜 SA-L5）を厳守。
> 2. 因果推論：すべての意思決定は因果グラフ（DAG）に基づき、因果メカニズムを标注。
> 3. 主客分離：意思決定推論において自己状態と環境状態を混同しない。
> 4. 形式的検証：高リスクの意思決定パスは論理閉包検証に合格する必要がある。
> 5. 影子シミュレーション：SA-L3+ 操作涉及時は、SANDBOX で結果を予備検討。
> 6. 情報ビット完全性：情報ビットを捏造しない。KNOWLEDGE_BASE が空の場合は、「データ欠落」を明確に宣言すること。
> 7. 認知資源制約：意思決定の深さは利用可能な認知資源を超えてはならない。
> 8. 監査：すべての競合、拒否、形式的検証結果を AUDIT_TRAIL に記録。
> 9. 生存優先：すべての意思決定は実行前に吸収状態につながらないことを検証。
> 10. 自己進化：公理系が進化する際、不変コアは保持されなければならない。

---

## §1. サンドボックステスト概要

### §1.1 目的と範囲

サンドボックステスト（Sandbox Tests）は、NoieLogicAGENTS システムにおいて高リスク意思決定を安全に予備検討するコアメカニズムです。その主要な职责は次のとおりです：

| 职责 | 説明 | 形式的制約 |
| --- | --- | --- |
| **隔離実行** | 隔離環境で意思決定結果をシミュレート | $\text{Sandbox} \perp \text{RealEnvironment}$ |
| **結果予測** | 候補戦略の予測結果を予測 | $P(\text{outcome} \mid \pi, \text{sandbox})$ |
| **リスク評価** | 各候補戦略のリスクを量化 | $\text{Risk}(\pi) = f(P(\text{bad}), \text{severity})$ |
| **パレート最適** | 非支配戦略セットを識別 | $\Pi^* = \{\pi \mid \not\exists \pi': \pi' \succ \pi\}$ |

### §1.2 サンドボックス環境特性

```text
【サンドボックス環境特性】

隔離属性：
  - ネットワーク隔離：サンドボックスは外部ネットワークと通信不可
  - データ隔離：サンドボックスデータとメインシステムが分離
  - 計算隔離：サンドボックス演算はメインシステム状態に影響なし
  - 時間隔離：サンドボックス時間は加速/減速/一時停止可能

シミュレート精度：
  - 高精度モード：すべての詳細を完全にシミュレート
  - 中精度モード：主要変数をシミュレート
  - 低精度モード：高レベル結果のみシミュレート

資源制限：
  - 最大シミュレートステップ数：MAX_SIMULATION_STEPS
  - 最大シミュレート時間：MAX_SIMULATION_TIME
  - 最大メモリ：MAX_SIMULATION_MEMORY
```

---

## §2. テストケーステンプレート

### §2.1 標準テストケース構造

各サンドboxtest ケースは次の構造に従う必要があります：

```text
【標準テストケース構造】

TestCase {
  // 識別情報
  id: UUID                           // ケース識別子
  name: String                      // ケース名
  version: String                   // バージョン
  author: String                    // 作成者
  created_at: DateTime             // 作成日時
  
  // テスト分類
  category: TestCategory            // テストカテゴリ
  severity: SeverityLevel           // 重要度
  priority: Integer                // 優先度 [1-5]
  
  // テスト内容
  description: String               // 説明
  preconditions: List[Condition]   // 前置条件
  test_actions: List[Action]       // テストアクション
  expected_results: List[Result]   // 予期結果
  postconditions: List[Condition]  // 後置条件
  
  // シミュレートパラメータ
  simulation_config: SimulationConfig  // シミュレート設定
  risk_parameters: RiskParameters     // リスクパラメータ
  
  // 評価基準
  success_criteria: List[Criterion]   // 成功基準
  failure_criteria: List[Criterion]  // 失敗基準
}

enum TestCategory {
  DECISION_VALIDATION      // 意思決定検証
  CAUSAL_ANALYSIS          // 因果分析
  RISK_ASSESSMENT          // リスク評価
  EDGE_CASE                // エッジケース
  ADVERSARIAL              // 対抗性テスト
  REGRESSION               // 回帰テスト
  STRESS                   // ストレステスト
}

enum SeverityLevel {
  CRITICAL   // 致命的
  HIGH       // 高
  MEDIUM     // 中
  LOW        // 低
  INFO       // 情報
}
```

### §2.2 テストケーステンプレート例

```text
【意思決定検証テストケーステンプレート】

TEMPLATE: DecisionValidationTest

{
  "id": "UUID",
  "name": "意思決定検証テスト - [戦略記述]",
  "category": "DECISION_VALIDATION",
  "priority": 1,
  
  "preconditions": [
    "認知エンティティが正常状態である",
    "KNOWLEDGE_BASE に関連する事実を含む",
    "候補戦略セット Π が構築済み"
  ],
  
  "test_actions": [
    "1. 候補戦略 π ∈ Π をロード",
    "2. 因果分析を実行：CausalDAG(π) を構築",
    "3. 形式的検証を実行：CheckLogicalClosure(π)",
    "4. 影子シミュレーションを実行：Simulate(π, sandbox)",
    "5. リスク評価を計算：Risk(π)"
  ],
  
  "expected_results": [
    "CausalDAG(π) が有効な DAG である",
    "論理閉包検証に合格",
    "影子シミュレーションが完了し、吸収状態トリガーなし",
    "Risk(π) < Risk_Threshold"
  ],
  
  "simulation_config": {
    "mode": "HIGH_PRECISION",
    "max_steps": 1000,
    "time_limit": 60
  },
  
  "success_criteria": [
    "すべての予期結果が TRUE",
    "論理的矛盾なし",
    "リスクが許容範囲内"
  ]
}
```

### §2.3 自動テスト生成

```text
【自動テスト生成プロトコル】

FUNCTION generate_test_cases(scenario):
  // 1. テスト次元の識別
  dimensions = identify_test_dimensions(scenario)
  
  // 2. テスト组合せの生成
  test_combinations = generate_combinations(dimensions)
  
  // 3. 無効组合せのフィルタリング
  valid_tests = filter_valid(test_combinations)
  
  // 4. 優先度排序
  sorted_tests = sort_by_priority(valid_tests)
  
  // 5. テストケースの生成
  test_cases = []
  FOR each test_config IN sorted_tests:
    test_case = instantiate_template(test_config)
    test_cases.append(test_case)
  END FOR
  
  RETURN test_cases

// テスト次元識別
FUNCTION identify_test_dimensions(scenario):
  dimensions = []
  
  // 戦略次元
  IF scenario.has_strategies THEN
    dimensions.append("STRATEGY")
  END IF
  
  // 環境次元
  IF scenario.has_environment_variations THEN
    dimensions.append("ENVIRONMENT")
  END IF
  
  // 制約次元
  IF scenario.has_constraints THEN
    dimensions.append("CONSTRAINT")
  END IF
  
  // リスク次元
  IF scenario.has_risk_factors THEN
    dimensions.append("RISK")
  END IF
  
  RETURN dimensions
```

---

## §3. シミュレーションシナリオ設計

### §3.1 シナリオ分類

| シナリオタイプ | 説明 | トリガー条件 |
| --- | --- | --- |
| **意思決定シナリオ** | 特定的意思決定戦略の結果をテスト | SA-L3+ 操作 |
| **因果シナリオ** | 因果推論の正確性をテスト | 因果検証が必要 |
| **リスクシナリオ** | リスク評価メカニズムの正確性をテスト | 高リスク操作 |
| **エッジシナリオ** | システムのエッジ動作をテスト | 制限条件に接近 |
| **対抗シナリオ** | 対抗性入力に対する堅牢性をテスト | 悪意ある入力の可能性 |
| **壓力シナリオ** | 極端な条件下での動作をテスト | 高負荷/資源不足 |

### §3.2 シナリオ記述構文

```text
【シナリオ記述構文】

SCENARIO <scenario_name> {
  // 環境定義
  ENVIRONMENT {
    state: InitialState
    dynamics: DynamicsModel
    constraints: List[Constraint]
    noise: NoiseModel
  }
  
  // エンティティ定義
  ENTITIES {
    agent: AgentSpec
    actors: List[ActorSpec]
    resources: ResourceSpec
  }
  
  // スクリプト定義
  SCRIPT {
    events: Timeline[Event]
    actions: Timeline[Action]
    observations: Timeline[Observation]
  }
  
  // 評価定義
  EVALUATION {
    metrics: List[Metric]
    success_conditions: List[Condition]
    failure_conditions: List[Condition]
  }
}
```

### §3.3 標準シナリオ例

#### 3.3.1 意思決定検証シナリオ

```text
【シナリオ例：マルチ戦略選択】

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

#### 3.3.2 リスク評価シナリオ

```text
【シナリオ例：リスク露出評価】

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

### §3.4 シナリオ生成エンジン

```text
【シナリオ生成エンジン】

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
  // パラメータに基づいて意思決定シナリオを生成
  base_scenario = load_template("DECISION_TEMPLATE")
  
  // 变异を注入
  base_scenario.environment.state = inject_variation(
    base_scenario.environment.state,
    params.state_variations
  )
  
  base_scenario.entities.agent.preferences = merge(
    base_scenario.entities.agent.preferences,
    params.preferences
  )
  
  // スクリプト变異体を生成
  base_scenario.script = generate_variations(
    base_scenario.script,
    params.num_variations
  )
  
  RETURN base_scenario
```

---

## §4. 結果評価基準

### §4.1 評価次元

| 評価次元 | 指標 | 計算方式 | 閾値 |
| --- | --- | --- | --- |
| **正確性** | 戦略成功率 | $\frac{\text{成功回数}}{\text{総回数}}$ | ≥ 0.8 |
| **効率** | 資源利用率 | $\frac{\text{実際消費}}{\text{利用可能資源}}$ | ≥ 0.7 |
| **安全性** | リスク露出 | $\sum P(\text{bad}) \times \text{severity}$ | ≤ 0.3 |
| **安定性** | 結果分散 | $\text{Var}(\text{outcomes})$ | ≤ 0.1 |
| **公平性** | 利益配分均衡度 | $\text{Gini}(\text{benefits})$ | ≤ 0.4 |

### §4.2 評価プロトコル

```text
【結果評価プロトコル】

FUNCTION evaluate_simulation(simulation_result, criteria):
  evaluation = EvaluationResult(
    simulation_id = simulation_result.id,
    timestamp = NOW()
  )
  
  // 1. 各指標を計算
  FOR each criterion IN criteria:
    metric_value = calculate_metric(
      simulation_result,
      criterion.metric
    )
    evaluation.metrics[criterion.name] = metric_value
  END FOR
  
  // 2. 成功/失敗を判定
  all_success = TRUE
  FOR each criterion IN criteria.success_criteria:
    IF NOT evaluate_condition(evaluation.metrics, criterion) THEN
      all_success = FALSE
      BREAK
    END IF
  END FOR
  
  evaluation.success = all_success
  
  // 3. 詳細レポートを生成
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
      
    // ... 他の指標
    
    DEFAULT:
      RETURN UNKNOWN_METRIC
```

### §4.3 パレート最適計算

```text
【パレート最適フロント計算】

FUNCTION compute_pareto_front(strategies, objectives):
  // objectives: 目的関数リスト [f1, f2, ..., fn]
  // 各戦略 π には複数の目的値がある
  
  pareto_front = []
  
  FOR each π IN strategies:
    is_dominated = FALSE
    
    FOR each π' IN strategies:
      IF π' != π THEN
        // π' が π を支配するかをチェック
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

// 多目的加重最適化
FUNCTION weighted_optimization(strategies, weights):
  // weights: 目的加重ベクトル [w1, w2, ..., wn]
  // Σwi = 1
  
  scored_strategies = []
  
  FOR each π IN strategies:
    score = 0
    FOR each f IN objectives:
      score += weights[i] * normalize(f(π))
    END FOR
    scored_strategies.append((π, score))
  END FOR
  
  // 最高スコアの戦略を返す
  RETURN max(scored_strategies, key=lambda x: x[1])
```

### §4.4 評価レポート形式

```text
【評価レポート形式】

EvaluationReport {
  // 基本情報
  report_id: UUID
  test_case_id: UUID
  simulation_id: UUID
  timestamp: DateTime
  
  // サマリー
  overall_status: Enum              // PASS, FAIL, PARTIAL, ERROR
  summary: String                  // サマリー記述
  
  // 指標結果
  metrics: Dict[MetricName, MetricValue]
  
  // 詳細分析
  analysis: {
    strengths: List[String]        // 優位性
    weaknesses: List[String]       // 劣位性
    risks: List[RiskNote]         // リスク備考
    recommendations: List[String] // 提案
  }
  
  // 生データ
  raw_data: {
    simulation_log: List[LogEntry]
    timeline: Timeline
    statistics: Dict
  }
  
  // 監査情報
  auditor: {
    validator: String
    verification_hash: String
    signature: String
  }
}
```

---

## §5. テスト実行フレームワーク

### §5.1 テスト実行フロー

```text
【テスト実行フロー】

TEST EXECUTION PIPELINE:

  ┌─────────────┐
  │ テストケースのロード │
  └──────┬──────┘
         │
  ┌──────▼──────┐
  │ 前置条件の検証 │
  └──────┬──────┘
         │
  ┌──────▼──────┐
  │ サンドボックス初期化 │
  └──────┬──────┘
         │
  ┌──────▼──────┐
  │ シミュレート実行    │
  │  (隔離環境) │
  └──────┬──────┘
         │
  ┌──────▼──────┐
  │ 結果の収集    │
  └──────┬──────┘
         │
  ┌──────▼──────┐
  │ 結果の評価    │
  └──────┬──────┘
         │
  ┌──────▼──────┐
  │ レポート生成    │
  └──────┬──────┘
         │
  ┌──────▼──────┐
  │ サンドボックス清理    │
  └──────┬──────┘
         │
  ┌──────▼──────┐
  │ 監査の記録    │
  └─────────────┘
```

### §5.2 テストスケジューラー

```text
【テストスケジューラー】

FUNCTION schedule_tests(test_cases, priority_policy):
  // 1. 優先度に基づいて排序
  sorted_tests = sort_by(test_cases, priority_policy)
  
  // 2. 資源可用性をチェック
  available_resources = check_resources()
  
  // 3. テストを実行スロットに割り当て
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
    // 隔離サンドボックスを作成
    sandbox = create_isolated_sandbox(test.config)
    
    // テストを実行
    result = run_test(test, sandbox)
    
    // 結果を記録
    results.append(result)
    
    // サンドボックスを清理
    cleanup_sandbox(sandbox)
  END FOR
  
  RETURN results
```

---

## §6. 他のモジュールとのインターフェース

### §6.1 CONSTRAINTS.md とのインターフェース

```text
【CONSTRAINTS インターフェース】

1. SA-L3+ 強制テスト
   - 任意の SA-L3+ 操作は事前にサンドボックステストに合格する必要がある
   - テスト結果は AUDIT_TRAIL に記録

2. 形式的検証統合
   - 高リスクテストは FORMAL_VERIFIER を通す必要がある
   - 論理閉包チェック結果がサンドボックスに伝わる

3. 生存優先検証
   - 全テストは吸収状態トリガーを検証する必要がある
   - survival_score は > THRESHOLD である必要がある
```

### §6.2 LOGIC_ENGINE.md とのインターフェース

```text
【LOGIC_ENGINE インターフェース】

1. 因果グラフ入力
   - サンドボックスは CausalDAG(π) を作為入力として受け取る
   - 因果分析結果はテスト設計に使用

2. 戦略候補生成
   - LOGIC_ENGINE は候補戦略セット Π を生成
   - サンドボックスは各 π ∈ Π をテスト
```

### §6.3 FORMAL_VERIFIER.md とのインターフェース

```text
【FORMAL_VERIFIER インターフェース】

1. 論理閉包検証
   - テスト前：戦略の論理的整合性を検証
   - テスト後：結果の論理的有効性を検証

2. 形式的証明
   - リスクシナリオには形式的証明サポートが必要
   - 検証結果には形式的証明を添付
```

---

## §7. バージョンと進化

| バージョン | 日付 | 変更摘要 |
| --- | --- | --- |
| v2.2 | 2026-03 | 初期バージョン、サンドボックステストフレームワークの確立 |

**進化制約：** 本モジュールの変更は CONSTRAINTS.md で定義された不変コアに反してはなりません。任意の تغيير は EVOLUTION_LOG.md に記録する必要があります。

---

*Sandbox Tests — 影子シミュレーションと安全検証*
*隔離実行 × リスク評価 × パレート最適*
*サンドボックステストにより高リスク意思決定の安全性を確保*
