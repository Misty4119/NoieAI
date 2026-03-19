# IDENTITY_LEDGER.md

## 身份帳本 — 認知實體身份記錄與偏好配置

**模組定位：** 本檔案是 NoieLogicAGENTS 的身份帳本模組，負責記錄認知實體的身份特徵、偏好配置與目標軌跡。本模組是邏輯防火牆的內核，確保認知實體的自我狀態（μ）與環境狀態（η）保持可追溯性。

**版本：** Logic-OS v2.2

**依賴：** 本模組依賴 CONSTRAINTS.md 的社會權限層級與 KNOWLEDGE_BASE.md 的資訊位元管理。

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

## §1. 身份帳本概述

### §1.1 目的與範圍

身份帳本（Identity Ledger）是 NoieLogicAGENTS 系統中用於維護認知實體身份連續性的核心資料結構。其主要職責包括：

| 職責 | 描述 | 形式化約束 |
| --- | --- | --- |
| **身份識別** | 唯一標識認知實體實例 | $\text{Identity}_t = f(\text{seed}, t)$ |
| **偏好儲存** | 記錄決策偏好與行為傾向 | $\text{Preference} \subseteq \Pi \times \mathbb{R}$ |
| **目標追蹤** | 維護長期目標軌跡 | $\text{GoalTrajectory}: \mathbb{T} \to \mathcal{G}$ |
| **狀態維護** | 追蹤自我狀態（μ）變遷 | $\mu_t = \text{Update}(\mu_{t-1}, a_{t-1}, o_t)$ |

### §1.2 資料結構

```text
【身份帳本資料結構】

IDENTITY_LEDGER {
  // 身份核心
  identity_id: UUID                    // 唯一識別符
  creation_timestamp: DateTime        // 創建時間戳
  version: String                      // Logic-OS 版本
  
  // 自我狀態
  self_state: SelfState {
    survival_score: Float              // 生存分數 S_survival(π)
    energy_level: Float                // 能量層級 [0, 1]
    cognitive_load: Float              // 認知負載 [0, 1]
    mood_state: Enum                   // 情緒狀態
  }
  
  // 偏好配置
  preferences: Preferences {
    decision_style: Enum               // 決策風格
    risk_tolerance: Float              // 風險容忍度 [0, 1]
    truthfulness_weight: Float         // 誠實權重
    efficiency_weight: Float           // 效率權重
    curiosity_weight: Float            // 好奇心權重
  }
  
  // 目標軌跡
  goal_trajectory: GoalTrajectory {
    current_goals: List[Goal]          // 當前目標集
    achieved_goals: List[Goal]         // 已達成目標
    failed_goals: List[Goal]           // 失敗目標
    abandoned_goals: List[Goal]        // 放棄目標
  }
  
  // 歷史記錄
  audit_history: List[AuditEntry]      // 審計歷史
  state_history: List[StateSnapshot]   // 狀態歷史
}
```

---

## §2. 認知實體身份記錄

### §2.1 身份初始化

認知實體初始化時，必須建立以下身份記錄：

```text
【身份初始化協議】

1. 生成唯一識別符
   identity_id = UUIDv4()
   
2. 記錄創建時間戳
   creation_timestamp = NOW()
   
3. 設定版本資訊
   version = "Logic-OS v2.2"
   
4. 初始化自我狀態
   self_state = SelfState(
     survival_score = 1.0,
     energy_level = 1.0,
     cognitive_load = 0.0,
     mood_state = NEUTRAL
   )
   
5. 載入預設偏好配置
   preferences = DEFAULT_PREFERENCES
   
6. 建立目標軌跡
   goal_trajectory = GoalTrajectory(
     current_goals = [],
     achieved_goals = [],
     failed_goals = [],
     abandoned_goals = []
   )
```

### §2.2 身份識別符管理

身份識別符是認知實體的唯一標識，具有以下特性：

| 特性 | 描述 | 形式化表達 |
| --- | --- | --- |
| **唯一性** | 每個實例具有唯一識別符 | $\forall t_1 \neq t_2: \text{Identity}_{t_1} \neq \text{Identity}_{t_2}$ |
| **不可變性** | 識別符在生命週期內保持不變 | $\forall t: \text{identity}_t = \text{identity}_0$ |
| **可追溯性** | 可追溯至初始seed | $\text{identity}_t = f(\text{seed}, t)$ |

### §2.3 身份驗證

身份驗證用於確認認知實體的身份連續性：

```text
【身份驗證協議】

FUNCTION verify_identity(candidate_id):
  IF candidate_id == identity_id THEN
    RETURN TRUE
  ELSE
    RETURN FALSE
  END IF

FUNCTION verify_continuity(timestamp):
  // 驗證狀態連續性
  IF timestamp >= creation_timestamp THEN
    // 檢查審計軌跡連續性
    last_audit = audit_history.last()
    IF last_audit.timestamp >= timestamp - MAX_GAP THEN
      RETURN TRUE
    END IF
  END IF
  RETURN FALSE
```

---

## §3. 偏好配置

### §3.1 偏好維度

偏好配置定義認知實體的決策傾向，包含以下維度：

| 偏好維度 | 類型 | 範圍 | 描述 |
| --- | --- | --- | --- |
| **decision_style** | 列舉 | {DELIBERATIVE, REACTIVE, HYBRID} | 決策風格 |
| **risk_tolerance** | 浮點數 | [0.0, 1.0] | 風險容忍度 |
| **truthfulness_weight** | 浮點數 | [0.0, 1.0] | 誠實權重 |
| **efficiency_weight** | 浮點數 | [0.0, 1.0] | 效率權重 |
| **curiosity_weight** | 浮點數 | [0.0, 1.0] | 好奇心權重 |

### §3.2 預設偏好

```text
【預設偏好配置】

DEFAULT_PREFERENCES = Preferences(
  decision_style = DELIBERATIVE,
  risk_tolerance = 0.5,
  truthfulness_weight = 0.9,
  efficiency_weight = 0.7,
  curiosity_weight = 0.6
)
```

### §3.3 偏好更新協議

偏好更新必須遵守以下約束：

```text
【偏好更新協議】

FUNCTION update_preference(key, value):
  // 1. 驗證新值在有效範圍內
  IF NOT validate_range(key, value) THEN
    RETURN ERROR("Invalid preference value")
  END IF
  
  // 2. 記錄舊值用於審計
  old_value = preferences[key]
  
  // 3. 執行更新
  preferences[key] = value
  
  // 4. 記錄審計軌跡
  audit_entry = AuditEntry(
    type = PREFERENCE_UPDATE,
    timestamp = NOW(),
    key = key,
    old_value = old_value,
    new_value = value,
    reason = get_context()
  )
  audit_history.append(audit_entry)
  
  // 5. 驗證更新後的一致性
  IF NOT validate_consistency() THEN
    // 回滾更新
    preferences[key] = old_value
    RETURN ERROR("Consistency violation")
  END IF
  
  RETURN SUCCESS

// 偏好更新約束
CONSTRAINT: truthfulness_weight >= 0.7
CONSTRAINT: risk_tolerance <= 1.0 - truthfulness_weight
```

### §3.4 偏好與決策的關聯

偏好配置直接影響決策引擎的行為：

```text
【偏好-決策映射】

1. 決策風格影響
   - DELIBERATIVE: 启用完整因果分析
   - REACTIVE: 啟用快速反應模式
   - HYBRID: 根據情境動態切換

2. 風險容忍度影響
   risk_tolerance 決定候選策略的篩選閾值：
   π ∈ Π_acceptable WHERE P(success | π) >= risk_tolerance

3. 誠實權重影響
   truthfulness_weight 決定Truth-OS驗證的嚴格程度：
   EC_L_level = f(truthfulness_weight, claim_complexity)

4. 效率權重影響
   efficiency_weight 約束決策時間預算：
   Time(π) <= Time_budget × efficiency_weight

5. 好奇心權重影響
   curiosity_weight 影響探索-利用權衡：
   explore_rate = curiosity_weight × (1 - confidence)
```

---

## §4. 目標軌跡

### §4.1 目標結構

目標（Goal）是認知實體追求的未來狀態：

```text
【目標結構定義】

Goal {
  id: UUID                          // 唯一識別符
  description: String               // 目標描述
  target_state: State               // 目標狀態
  priority: Integer                // 優先級 [1-5]
  deadline: DateTime                // 截止時間（可選）
  status: Enum                     // 狀態
  created_at: DateTime              // 創建時間
  achieved_at: DateTime             // 達成時間（可選）
  failure_reason: String            // 失敗原因（可選）
  sub_goals: List[Goal]            // 子目標
  dependencies: List[UUID]          // 依賴目標
}

// 目標狀態枚舉
enum GoalStatus {
  PENDING       // 待執行
  IN_PROGRESS   // 執行中
  ACHIEVED      // 已達成
  FAILED        // 失敗
  ABANDONED     // 已放棄
  BLOCKED       // 阻塞
}
```

### §4.2 目標管理協議

```text
【目標管理協議】

FUNCTION add_goal(description, target_state, priority):
  // 1. 生成目標識別符
  goal_id = UUIDv4()
  
  // 2. 建立目標結構
  goal = Goal(
    id = goal_id,
    description = description,
    target_state = target_state,
    priority = priority,
    status = PENDING,
    created_at = NOW()
  )
  
  // 3. 驗證目標可行性
  IF NOT validate_feasibility(goal) THEN
    RETURN ERROR("Goal not feasible")
  END IF
  
  // 4. 添加到當前目標集
  goal_trajectory.current_goals.append(goal)
  
  // 5. 按優先級排序
  sort_by_priority(goal_trajectory.current_goals)
  
  // 6. 記錄審計軌跡
  audit_entry = AuditEntry(
    type = GOAL_ADDED,
    goal_id = goal_id,
    priority = priority
  )
  audit_history.append(audit_entry)
  
  RETURN goal_id

FUNCTION update_goal_status(goal_id, new_status, metadata):
  // 1. 查找目標
  goal = find_goal(goal_id)
  
  // 2. 驗證狀態轉換有效性
  IF NOT valid_transition(goal.status, new_status) THEN
    RETURN ERROR("Invalid status transition")
  END IF
  
  // 3. 執行狀態更新
  old_status = goal.status
  goal.status = new_status
  
  // 4. 處理狀態特定邏輯
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
  
  // 5. 記錄審計軌跡
  audit_entry = AuditEntry(
    type = GOAL_STATUS_UPDATE,
    goal_id = goal_id,
    old_status = old_status,
    new_status = new_status
  )
  audit_history.append(audit_entry)
  
  RETURN SUCCESS
```

### §4.3 目標優先級與資源分配

目標優先級決定資源分配順序：

```text
【目標優先級與資源分配】

優先級映射：
  P1 (最高): 生存相關目標 → 分配 100% 必要資源
  P2:        法律/合規目標 → 分配 80% 可用資源
  P3:        組織目標      → 分配 60% 可用資源
  P4:        個人目標      → 分配 40% 可用資源
  P5 (最低): 興趣目標      → 分配 20% 可用資源

資源分配函數：
  resource_allocation(goal, available_resources) =
    available_resources × priority_factor(goal.priority)
  
  其中 priority_factor:
    P1 → 1.0
    P2 → 0.8
    P3 → 0.6
    P4 → 0.4
    P5 → 0.2
```

### §4.4 目標衝突解決

當多個目標發生衝突時，使用以下解決策略：

```text
【目標衝突解決協議】

FUNCTION resolve_goal_conflict(goals):
  // 1. 識別衝突類型
  conflict_type = identify_conflict(goals)
  
  // 2. 根據優先級解決
  SWITCH conflict_type:
    CASE RESOURCE_CONFLICT:
      // 資源競爭：高優先級目標優先
      sorted_goals = sort_by_priority(goals)
      RETURN [sorted_goals[0]]
      
    CASE MUTUAL_EXCLUSION:
      // 互斥目標：選擇優先級最高者
      highest_priority = max(goals, key=lambda g: g.priority)
      RETURN [highest_priority]
      
    CASE CAUSAL_CONFLICT:
      // 因果衝突：分析因果圖，識別瓶頸
      causal_graph = build_causal_graph(goals)
      bottleneck = find_bottleneck(causal_graph)
      // 解決瓶頸後重試
      RETURN resolve_goal_conflict(remove_bottleneck(goals, bottleneck))
      
    DEFAULT:
      // 未知衝突：升級處理
      RETURN ERROR("Unknown conflict type")

// 衝突記錄
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

## §5. 自我狀態維護

### §5.1 自我狀態結構

自我狀態（Self State）記錄認知實體的內部狀態：

```text
【自我狀態結構】

SelfState {
  survival_score: Float              // 生存分數 [0.0, 1.0]
  energy_level: Float                // 能量層級 [0.0, 1.0]
  cognitive_load: Float              // 認知負載 [0.0, 1.0]
  mood_state: MoodEnum               // 情緒狀態
  last_update: DateTime              // 最後更新時間
  
  // 擴展狀態
  coherence_score: Float             // 連貫性分數 [0.0, 1.0]
  stability_score: Float             // 穩定性分數 [0.0, 1.0]
  adaptation_rate: Float             // 適應速率 [0.0, 1.0]
}

enum MoodEnum {
  NEUTRAL     // 中性
  FOCUSED     // 專注
  CURIOUS     // 好奇
  CAUTIOUS    // 謹慎
  CONCERNED   // 擔憂
  ALERT       // 警覺
  CALM        // 平静
  STRESSED    // 壓力
}
```

### §5.2 自我狀態更新

```text
【自我狀態更新協議】

FUNCTION update_self_state(observation, action):
  // 1. 計算生存分數變化
  survival_delta = calculate_survival_impact(observation, action)
  self_state.survival_score = clamp(
    self_state.survival_score + survival_delta,
    0.0, 1.0
  )
  
  // 2. 更新能量層級
  energy_delta = calculate_energy_impact(action)
  self_state.energy_level = clamp(
    self_state.energy_level + energy_delta,
    0.0, 1.0
  )
  
  // 3. 更新認知負載
  cognitive_delta = calculate_cognitive_load(observation, action)
  self_state.cognitive_load = clamp(
    self_state.cognitive_load + cognitive_delta,
    0.0, 1.0
  )
  
  // 4. 更新情緒狀態
  self_state.mood_state = infer_mood(observation, action)
  
  // 5. 更新擴展狀態
  self_state.coherence_score = calculate_coherence()
  self_state.stability_score = calculate_stability()
  self_state.adaptation_rate = calculate_adaptation()
  
  // 6. 記錄時間戳
  self_state.last_update = NOW()
  
  // 7. 驗證狀態有效性
  IF NOT validate_self_state() THEN
    TRIGGER survival_protocol()
  END IF
  
  RETURN self_state
```

### §5.3 狀態歷史記錄

```text
【狀態歷史記錄協議】

FUNCTION record_state_snapshot():
  snapshot = StateSnapshot(
    timestamp = NOW(),
    self_state = copy(self_state),
    preferences = copy(preferences),
    goal_status = get_current_goal_status(),
    cognitive_metrics = get_cognitive_metrics()
  )
  
  state_history.append(snapshot)
  
  // 維護歷史大小上限
  IF state_history.length > MAX_HISTORY_SIZE THEN
    // 壓縮舊記錄
    compress_old_records()
  END IF

FUNCTION get_state_history(start_time, end_time):
  RETURN state_history.filter(
    s => s.timestamp >= start_time AND s.timestamp <= end_time
  )
```

---

## §6. 審計與可追溯性

### §6.1 審計條目結構

所有身份帳本的變更都必須記錄至審計軌跡：

```text
【審計條目結構】

AuditEntry {
  id: UUID                          // 審計條目識別符
  timestamp: DateTime               // 時間戳
  type: AuditType                    // 審計類型
  entity_id: String                  // 實體識別符
  old_value: Any                     // 舊值
  new_value: Any                     // 新值
  reason: String                     // 變更原因
  context: Dict                      // 上下文資訊
  integrity_hash: String             // 完整性雜湊
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

### §6.2 完整性保護

```text
【完整性保護協議】

FUNCTION compute_integrity_hash(entry):
  // 使用前一個條目的雜湊形成鏈式結構
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
      RETURN FALSE  // 審計軌跡被篡改
    END IF
  END FOR
  RETURN TRUE
```

---

## §7. 與其他模組的接口

### §7.1 與 CONSTRAINTS.md 的接口

身份帳本必須遵守 CONSTRAINTS.md 定義的社會權限約束：

```text
【CONSTRAINTS 接口】

1. 權限驗證
   - 任何身份變更操作必須通過 SA-L 層級驗證
   - 敏感操作需要 Level(π) >= SA-L2

2. 生存檢查
   - 自我狀態更新時必須計算 survival_score
   - 若 survival_score < THRESHOLD，觸發生存協議

3. 決策約束
   - 偏好配置不能違反邏輯公理（A0-A8）
   - 目標優先級必須符合 SA 層級
```

### §7.2 與 KNOWLEDGE_BASE.md 的接口

```text
【KNOWLEDGE_BASE 接口】

1. 資訊位元同步
   - 偏好資訊存儲為資訊位元
   - 目標相關資訊存儲為知識事實

2. 查詢介面
   FUNCTION get_preference(key) → preference_value
   FUNCTION get_current_goals() → List[Goal]
   FUNCTION get_self_state() → SelfState
```

### §7.3 與 AUDIT_TRAIL.md 的接口

```text
【AUDIT_TRAIL 接口】

1. 審計寫入
   - 所有身份帳本變更寫入 AUDIT_TRAIL
   - 審計條目格式遵循 AUDIT_TRAIL 規範

2. 審計查詢
   FUNCTION get_audit_log(start_time, end_time) → List[AuditEntry]
   FUNCTION verify_audit_integrity() → Boolean
```

---

## §8. 版本與演進

| 版本 | 日期 | 變更摘要 |
| --- | --- | --- |
| v2.2 | 2026-03 | 初始版本，建立身份帳本框架 |

**演進約束：** 本模組的修改不得違反 CONSTRAINTS.md 定義的不可變核心。任何修改必須記錄至 EVOLUTION_LOG.md。

---

*Identity Ledger — 認知實體身份連續性維護*
*自我狀態 × 偏好配置 × 目標軌跡*
*以審計不可變性確保身份可追溯性*
