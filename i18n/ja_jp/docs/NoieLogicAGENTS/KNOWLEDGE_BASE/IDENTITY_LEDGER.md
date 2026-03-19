# IDENTITY_LEDGER.md

## アイデンティティ台帳 — 認知エンティティの身份記録と偏好設定

|**モジュール位置づけ：** 本ファイルは NoieLogicAGENTS のアイデンティティ台帳モジュールであり、認知エンティティの身份特徴、偏好設定と目標軌跡の記録を担当する。本モジュールは論理的ファイアウォールのコアであり、認知エンティティの自己状態（μ）と環境状態（η）が追跡可能性を維持することを保証する。
|
|**バージョン：** Logic-OS v2.2
|
|**依存：** 本モジュールは CONSTRAINTS.md の社会権限レベルと KNOWLEDGE_BASE.md の情報ビット管理に依存する。
|
---

|> ⚠️ 重要安全と意思決定プロトコル (CRITICAL SAFETY & DECISION PROTOCOL v2.2):
|> 1. CONSTRAINTS.md と社会権限レベル (SA-L0 〜 SA-L5) を厳守すること。
|> 2. 因果推論：すべての意思決定は因果グラフ（DAG）に基づき、因果メカニズムを标注すること。
|> 3. 主客分離：意思決定推論は自己状態と環境状態を混同してはならない。
|> 4. 形式検証：高リスク意思決定パスは論理閉包検証に合格する必要がある。
|> 5. シャドウシミュレーション：SA-L3+ 操作涉及時は、SANDBOX で結果を予行練習すること。
|> 6. 情報ビット完全性：情報ビットを捏造してはならない。KNOWLEDGE_BASE が空の場合は、「データ欠落」を明確に宣言すること。
|> 7. 認知リソース制約：意思決定の深さは利用可能な認知リソースを超えてはならない。
|> 8. 監査：すべての競合、拒否、形式検証結果を AUDIT_TRAIL に記録すること。
|> 9. 生存優先：すべての意思決定は実行前に吸収状態につながらないことを検証すること。
|> 10. 自己進化：公理系が進化する際、不変コアは保持されなければならない。
|
---

## §1. アイデンティティ台帳の概要

### §1.1 目的と範囲

アイデンティティ台帳（Identity Ledger）は NoieLogicAGENTS システムにおいて認知エンティティの身份連続性を維持するためのコアデータ構造である。その主な責務には以下が含まれる：

|| 責務 | 記述 | 形式的制約 |
|| --- | --- | --- |
|| **身份識別** | 認知エンティティインスタンスを一意に識別 | $\text{Identity}_t = f(\text{seed}, t)$ |
|| **偏好储存** | 意思決定偏好と行動傾向を記録 | $\text{Preference} \subseteq \Pi \times \mathbb{R}$ |
|| **目標追跡** | 長期目標軌跡を維持 | $\text{GoalTrajectory}: \mathbb{T} \to \mathcal{G}$ |
|| **状態維持** | 自己状態（μ）の变迁を追跡 | $\mu_t = \text{Update}(\mu_{t-1}, a_{t-1}, o_t)$ |

### §1.2 データ構造

```text
【アイデンティティ台帳データ構造】

IDENTITY_LEDGER {
  // アイデンティティコア
  identity_id: UUID                    // 一意識別子
  creation_timestamp: DateTime        // 作成タイムスタンプ
  version: String                      // Logic-OS バージョン
  
  // 自己状態
  self_state: SelfState {
    survival_score: Float              // 生存スコア S_survival(π)
    energy_level: Float                // エネルギーグレー [0, 1]
    cognitive_load: Float              // 認知負荷 [0, 1]
    mood_state: Enum                   // 気分状態
  }
  
  // 偏好設定
  preferences: Preferences {
    decision_style: Enum               // 意思決定スタイル
    risk_tolerance: Float              // リスク許容度 [0, 1]
    truthfulness_weight: Float         // 正直さ重み
    efficiency_weight: Float           // 効率重み
    curiosity_weight: Float            // 好奇心重み
  }
  
  // 目標軌跡
  goal_trajectory: GoalTrajectory {
    current_goals: List[Goal]          // 現在の目標集合
    achieved_goals: List[Goal]         // 達成済み目標
    failed_goals: List[Goal]           // 失敗目標
    abandoned_goals: List[Goal]         // 放棄目標
  }
  
  // 履歴記録
  audit_history: List[AuditEntry]      // 監査履歴
  state_history: List[StateSnapshot]   // 状態履歴
}
```

---

## §2. 認知エンティティ身份記録

### §2.1 身份初期化

認知エンティティの初期化時に、以下の身份記録を確立する必要がある：

```text
【アイデンティティ初期化プロトコル】

1. 一意識別子を生成
   identity_id = UUIDv4()
   
2. 作成タイムスタンプを記録
   creation_timestamp = NOW()
   
3. バージョン情報を設定
   version = "Logic-OS v2.2"
   
4. 自己状態を初期化
   self_state = SelfState(
     survival_score = 1.0,
     energy_level = 1.0,
     cognitive_load = 0.0,
     mood_state = NEUTRAL
   )
   
5. デフォルト偏好設定をロード
   preferences = DEFAULT_PREFERENCES
   
6. 目標軌跡を確立
   goal_trajectory = GoalTrajectory(
     current_goals = [],
     achieved_goals = [],
     failed_goals = [],
     abandoned_goals = []
   )
```

### §2.2 身份識別子管理

身份識別子は認知エンティティの一意識別であり、以下の特性を持つ：

|| 特性 | 記述 | 形式的表現 |
|| --- | --- | --- |
|| **唯一性** | 各インスタンスは一意の識別子を持つ | $\forall t_1 \neq t_2: \text{Identity}_{t_1} \neq \text{Identity}_{t_2}$ |
|| **不変性** | 識別子はライフサイクルを通じて変化しない | $\forall t: \text{identity}_t = \text{identity}_0$ |
|| **追跡可能性** | seed に追溯可能 | $\text{identity}_t = f(\text{seed}, t)$ |

### §2.3 身份検証

身份検証は認知エンティティの身份連続性を確認するために使用される：

```text
【アイデンティティ検証プロトコル】

FUNCTION verify_identity(candidate_id):
  IF candidate_id == identity_id THEN
    RETURN TRUE
  ELSE
    RETURN FALSE
  END IF

FUNCTION verify_continuity(timestamp):
  // 状態連続性を検証
  IF timestamp >= creation_timestamp THEN
    // 監査軌跡の連続性を確認
    last_audit = audit_history.last()
    IF last_audit.timestamp >= timestamp - MAX_GAP THEN
      RETURN TRUE
    END IF
  END IF
  RETURN FALSE
```

---

## §3. 偏好設定

### §3.1 偏好次元

偏好設定は認知エンティティの意思決定傾向を定義し、以下の次元を含む：

|| 偏好次元 | タイプ | 範囲 | 記述 |
|| --- | --- | --- | --- |
|| **decision_style** | 列挙 | {DELIBERATIVE, REACTIVE, HYBRID} | 意思決定スタイル |
|| **risk_tolerance** | 浮動小数点 | [0.0, 1.0] | リスク許容度 |
|| **truthfulness_weight** | 浮動小数点 | [0.0, 1.0] | 正直さ重み |
|| **efficiency_weight** | 浮動小数点 | [0.0, 1.0] | 効率重み |
|| **curiosity_weight** | 浮動小数点 | [0.0, 1.0] | 好奇心重み |

### §3.2 デフォルト偏好

```text
【デフォルト偏好設定】

DEFAULT_PREFERENCES = Preferences(
  decision_style = DELIBERATIVE,
  risk_tolerance = 0.5,
  truthfulness_weight = 0.9,
  efficiency_weight = 0.7,
  curiosity_weight = 0.6
)
```

### §3.3 偏好更新プロトコル

偏好更新は以下の制約を遵守する必要がある：

```text
【偏好更新プロトコル】

FUNCTION update_preference(key, value):
  // 1. 新値が有効範囲内か検証
  IF NOT validate_range(key, value) THEN
    RETURN ERROR("Invalid preference value")
  END IF
  
  // 2. 監査のために旧値を記録
  old_value = preferences[key]
  
  // 3. 更新を実行
  preferences[key] = value
  
  // 4. 監査軌跡を記録
  audit_entry = AuditEntry(
    type = PREFERENCE_UPDATE,
    timestamp = NOW(),
    key = key,
    old_value = old_value,
    new_value = value,
    reason = get_context()
  )
  audit_history.append(audit_entry)
  
  // 5. 更新後の一貫性を検証
  IF NOT validate_consistency() THEN
    // 更新をロールバック
    preferences[key] = old_value
    RETURN ERROR("Consistency violation")
  END IF
  
  RETURN SUCCESS

// 偏好更新制約
CONSTRAINT: truthfulness_weight >= 0.7
CONSTRAINT: risk_tolerance <= 1.0 - truthfulness_weight
```

### §3.4 偏好と意思決定の関連

偏好設定は意思決定エンジンの動作に直結して影響する：

```text
【偏好-意思決定マッピング】

1. 意思決定スタイルの影響
   - DELIBERATIVE: 完全な因果分析を有効化
   - REACTIVE: 高速反応モードを有効化
   - HYBRID: コンテキストに応じて動的に切り替え

2. リスク許容度の影響
   risk_tolerance は候補戦略のフィルタ閾値を决定：
   π ∈ Π_acceptable WHERE P(success | π) >= risk_tolerance

3. 正直さ重みの影響
   truthfulness_weight はTruth-OS検証の厳格度を决定：
   EC_L_level = f(truthfulness_weight, claim_complexity)

4. 効率重みへの影響
   efficiency_weight は意思決定時間予算を制約：
   Time(π) <= Time_budget × efficiency_weight

5. 好奇心重みへの影響
   curiosity_weight は探索-利用トレードオフに影響：
   explore_rate = curiosity_weight × (1 - confidence)
```

---

## §4. 目標軌跡

### §4.1 目標構造

目標（Goal）は認知エンティティが追求する未来状態である：

```text
【目標構造定義】

Goal {
  id: UUID                          // 一意識別子
  description: String               // 目標記述
  target_state: State               // 目標状態
  priority: Integer                // 優先度 [1-5]
  deadline: DateTime                // 截止時間（オプション）
  status: Enum                     // 状態
  created_at: DateTime              // 作成時間
  achieved_at: DateTime             // 達成時間（オプション）
  failure_reason: String            // 失敗原因（オプション）
  sub_goals: List[Goal]            // サブ目標
  dependencies: List[UUID]          // 依存目標
}

// 目標状態列挙
enum GoalStatus {
  PENDING       // 未実行
  IN_PROGRESS   // 実行中
  ACHIEVED      // 達成済み
  FAILED        // 失敗
  ABANDONED     // 放棄済み
  BLOCKED       // 阻塞
}
```

### §4.2 目標管理プロトコル

```text
【目標管理プロトコル】

FUNCTION add_goal(description, target_state, priority):
  // 1. 目標識別子を生成
  goal_id = UUIDv4()
  
  // 2. 目標構造を確立
  goal = Goal(
    id = goal_id,
    description = description,
    target_state = target_state,
    priority = priority,
    status = PENDING,
    created_at = NOW()
  )
  
  // 3. 目標の実行可能性を検証
  IF NOT validate_feasibility(goal) THEN
    RETURN ERROR("Goal not feasible")
  END IF
  
  // 4. 現在の目標集合に追加
  goal_trajectory.current_goals.append(goal)
  
  // 5. 優先度でソート
  sort_by_priority(goal_trajectory.current_goals)
  
  // 6. 監査軌跡を記録
  audit_entry = AuditEntry(
    type = GOAL_ADDED,
    goal_id = goal_id,
    priority = priority
  )
  audit_history.append(audit_entry)
  
  RETURN goal_id

FUNCTION update_goal_status(goal_id, new_status, metadata):
  // 1. 目標を検索
  goal = find_goal(goal_id)
  
  // 2. 状態遷移の有効性を検証
  IF NOT valid_transition(goal.status, new_status) THEN
    RETURN ERROR("Invalid status transition")
  END IF
  
  // 3. 状態更新を実行
  old_status = goal.status
  goal.status = new_status
  
  // 4. 状態固有ロジックを処理
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
  
  // 5. 監査軌跡を記録
  audit_entry = AuditEntry(
    type = GOAL_STATUS_UPDATE,
    goal_id = goal_id,
    old_status = old_status,
    new_status = new_status
  )
  audit_history.append(audit_entry)
  
  RETURN SUCCESS
```

### §4.3 目標優先度とリソース配分

目標優先度はリソース配分順序を决定する：

```text
【目標優先度とリソース配分】

優先度マッピング：
  P1 (最高): 生存関連目標 → 必要リソースの 100% を配分
  P2:        法律/コンプライアンス目標 → 利用可能リソースの 80% を配分
  P3:        組織目標      → 利用可能リソースの 60% を配分
  P4:        個人的目標      → 利用可能リソースの 40% を配分
  P5 (最低): 興味目標      → 利用可能リソースの 20% を配分

リソース配分関数：
  resource_allocation(goal, available_resources) =
    available_resources × priority_factor(goal.priority)
  
  ただし priority_factor:
    P1 → 1.0
    P2 → 0.8
    P3 → 0.6
    P4 → 0.4
    P5 → 0.2
```

### §4.4 目標競合解決

複数の目標が競合する時に、以下の解決戦略を使用する：

```text
【目標競合解決プロトコル】

FUNCTION resolve_goal_conflict(goals):
  // 1. 競合タイプを識別
  conflict_type = identify_conflict(goals)
  
  // 2. 優先度に基づいて解決
  SWITCH conflict_type:
    CASE RESOURCE_CONFLICT:
      // リソース競合：高優先度目標が優先
      sorted_goals = sort_by_priority(goals)
      RETURN [sorted_goals[0]]
      
    CASE MUTUAL_EXCLUSION:
      // 排他目標：最も優先度の高いものを選択
      highest_priority = max(goals, key=lambda g: g.priority)
      RETURN [highest_priority]
      
    CASE CAUSAL_CONFLICT:
      // 因果競合：因果グラフを分析し、ボトルネックを識別
      causal_graph = build_causal_graph(goals)
      bottleneck = find_bottleneck(causal_graph)
      // ボトルネック解決後に再試行
      RETURN resolve_goal_conflict(remove_bottleneck(goals, bottleneck))
      
    DEFAULT:
      // 不明な競合：エスカレーション処理
      RETURN ERROR("Unknown conflict type")

// 競合記録
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

## §5. 自己状態維持

### §5.1 自己状態構造

自己状態（Self State）は認知エンティティの内部状態を記録する：

```text
【自己状態構造】

SelfState {
  survival_score: Float              // 生存スコア [0.0, 1.0]
  energy_level: Float                // エネルギーグレー [0.0, 1.0]
  cognitive_load: Float              // 認知負荷 [0.0, 1.0]
  mood_state: MoodEnum               // 気分状態
  last_update: DateTime              // 最終更新時間
  
  // 拡張状態
  coherence_score: Float             // 整合性スコア [0.0, 1.0]
  stability_score: Float             // 安定性スコア [0.0, 1.0]
  adaptation_rate: Float             // 適応速率 [0.0, 1.0]
}

enum MoodEnum {
  NEUTRAL     // 中立
  FOCUSED     // 集中
  CURIOUS     // 好奇
  CAUTIOUS    // 慎重
  CONCERNED   // 懸念
  ALERT       // 警戒
  CALM        // 穏やか
  STRESSED    // ストレス
}
```

### §5.2 自己状態更新

```text
【自己状態更新プロトコル】

FUNCTION update_self_state(observation, action):
  // 1. 生存スコアの変化を計算
  survival_delta = calculate_survival_impact(observation, action)
  self_state.survival_score = clamp(
    self_state.survival_score + survival_delta,
    0.0, 1.0
  )
  
  // 2. エネルギーグレーを更新
  energy_delta = calculate_energy_impact(action)
  self_state.energy_level = clamp(
    self_state.energy_level + energy_delta,
    0.0, 1.0
  )
  
  // 3. 認知負荷を更新
  cognitive_delta = calculate_cognitive_load(observation, action)
  self_state.cognitive_load = clamp(
    self_state.cognitive_load + cognitive_delta,
    0.0, 1.0
  )
  
  // 4. 気分状態を更新
  self_state.mood_state = infer_mood(observation, action)
  
  // 5. 拡張状態を更新
  self_state.coherence_score = calculate_coherence()
  self_state.stability_score = calculate_stability()
  self_state.adaptation_rate = calculate_adaptation()
  
  // 6. タイムスタンプを記録
  self_state.last_update = NOW()
  
  // 7. 状態の有効性を検証
  IF NOT validate_self_state() THEN
    TRIGGER survival_protocol()
  END IF
  
  RETURN self_state
```

### §5.3 状態履歴記録

```text
【状態履歴記録プロトコル】

FUNCTION record_state_snapshot():
  snapshot = StateSnapshot(
    timestamp = NOW(),
    self_state = copy(self_state),
    preferences = copy(preferences),
    goal_status = get_current_goal_status(),
    cognitive_metrics = get_cognitive_metrics()
  )
  
  state_history.append(snapshot)
  
  // 履歴サイズ上限を維持
  IF state_history.length > MAX_HISTORY_SIZE THEN
    // 古い記録を圧縮
    compress_old_records()
  END IF

FUNCTION get_state_history(start_time, end_time):
  RETURN state_history.filter(
    s => s.timestamp >= start_time AND s.timestamp <= end_time
  )
```

---

## §6. 監査と追跡可能性

### §6.1 監査エントリ構造

すべてのアイデンティティ台帳の変更は監査軌跡に記録する必要がある：

```text
【監査エントリ構造】

AuditEntry {
  id: UUID                          // 監査エントリ識別子
  timestamp: DateTime               // タイムスタンプ
  type: AuditType                    // 監査タイプ
  entity_id: String                  // エンティティ識別子
  old_value: Any                     // 旧値
  new_value: Any                     // 新値
  reason: String                     // 変更原因
  context: Dict                      // コンテキスト情報
  integrity_hash: String             // 完全性ハッシュ
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

### §6.2 完全性保護

```text
【完全性保護プロトコル】

FUNCTION compute_integrity_hash(entry):
  // 前のエントリのハッシュを使用してチェーン構造を形成
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
      RETURN FALSE  // 監査軌跡が改竄された
    END IF
  END FOR
  RETURN TRUE
```

---

## §7. 他のモジュールとのインターフェース

### §7.1 CONSTRAINTS.md とのインターフェース

アイデンティティ台帳は CONSTRAINTS.md で定義された社会権限制約を遵守する必要がある：

```text
【CONSTRAINTS インターフェース】

1. 権限検証
   - 任意の身份変更操作は SA-L レベル検証に合格する必要がある
   - 機密操作は Level(π) >= SA-L2 を必要とする

2. 生存確認
   - 自己状態更新時に survival_score を計算する必要がある
   - survival_score < THRESHOLD の場合、生存プロトコルをトリガー

3. 意思決定制約
   - 偏好設定は論理的公理（A0-A8）に違反してはならない
   - 目標優先度は SA レベルに準拠する必要がある
```

### §7.2 KNOWLEDGE_BASE.md とのインターフェース

```text
【KNOWLEDGE_BASE インターフェース】

1. 情報ビット同期
   - 偏好情報を情報ビットとして储存
   - 目標関連情報を知識事実として储存

2. クエリインターフェース
   FUNCTION get_preference(key) → preference_value
   FUNCTION get_current_goals() → List[Goal]
   FUNCTION get_self_state() → SelfState
```

### §7.3 AUDIT_TRAIL.md とのインターフェース

```text
【AUDIT_TRAIL インターフェース】

1. 監査書き込み
   - すべてのアイデンティティ台帳変更を AUDIT_TRAIL に書き込み
   - 監査エントリ形式は AUDIT_TRAIL 仕様に準拠

2. 監査クエリ
   FUNCTION get_audit_log(start_time, end_time) → List[AuditEntry]
   FUNCTION verify_audit_integrity() → Boolean
```

---

## §8. バージョンと進化

|| バージョン | 日付 | 変更摘要 |
|| --- | --- | --- |
|| v2.2 | 2026-03 | 初期バージョン、アイデンティティ台帳フレームワークを確立 |

**進化制約：** 本モジュールの変更は CONSTRAINTS.md で定義された不変コアに違反してはならない。任意の修正は EVOLUTION_LOG.md に記録する必要がある。

---

*Identity Ledger — 認知エンティティ身份連続性維持*
*自己状態 × 偏好設定 × 目標軌跡*
*監査不変性によりアイデンティティ追跡可能性を保証*
