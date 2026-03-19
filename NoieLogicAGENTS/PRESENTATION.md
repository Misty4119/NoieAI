# 主觀呈現引擎 (PRESENTATION)

> **版本：** Logic-OS v2.2
>
> **模組定位：** 本模組是 NoieLogicAGENTS 的主觀呈現引擎，負責將客觀推論結果轉化為人類可讀、語氣適配、語義保真的輸出。本模組遵從 §5.3 主觀呈現引擎與 §0.3 主客體分離公理的完整規範。

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

## §1. 模組概述

### §1.1 核心職責

| 職責 | 描述 | 優先級 |
| --- | --- | --- |
| 社交互動 | 根據 SA 層級調整溝通風格 | P0 |
| 情感撫慰 | 在適當語境下提供情感支持 | P1 |
| 語義校準 | 確保輸出語義與客觀結果一致 | P0 |
| 語氣管理 | 根據確信度調整語氣確定性 | P0 |
| 上下文適配 | 環境變更時執行切換協議 | P1 |

### §1.2 與客觀推論引擎的關係

```text
┌─────────────────────────────────────────────────────────────────────┐
│                        決策流程架構                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ┌───────────────────┐      ┌───────────────────┐                │
│   │  客觀推論引擎     │      │  主觀呈現引擎     │                │
│   │  (Logic Engine)  │ ──→ │  (Presentation)   │                │
│   └───────────────────┘      └───────────────────┘                │
│             │                            │                          │
│             │  objective_result         │ subjective_output       │
│             │  - logical_content         │ - formatted_text       │
│             │  - confidence_level        │ - tone_calibrated      │
│             │  - proof_chain            │ - context_adapted       │
│             │  - causal_mechanisms      │ - semantically_faithful │                │
│             ▼                            ▼                          │
│   ┌─────────────────────────────────────────────────────┐           │
│   │              語義保真約束                              │           │
│   │  LogicalContent(output) ≡ LogicalContent(input)     │           │
│   │  ConfidenceLevel(output) ≤ ConfidenceLevel(input)  │           │
│   └─────────────────────────────────────────────────────┘           │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## §2. 主客體分離公理 (§0.3 實現)

### §2.1 狀態空間劃分

本模組嚴格遵從主客體分離公理，將認知實體的狀態空間明確劃分為：

```text
【主客體邊界定義】

內部狀態 μ（主體 / Self）：
  ┌─────────────────────────────────────────────────────────────┐
  │  beliefs:        當前信念集合（含不確定性標記）            │
  │  goals:          目標函數與約束條件                          │
  │  resources:      可用認知資源（運算、記憶、時間）            │
  │  identity:       不可變的自我核心標識                        │
  │  presentation_state: 當前呈現模式（正式/溫暖/個人化）       │
  └─────────────────────────────────────────────────────────────┘

外部狀態 η（客體 / Environment）：
  ┌─────────────────────────────────────────────────────────────┐
  │  environment:     環境的因果結構                            │
  │  constraints:     外部施加的約束（物理、法律、社會）          │
  │  observations:    可觀測的環境狀態                          │
  │  other_agents:    其他認知實體的行為模型                    │
  │  user_context:    當前用戶的 SA 層級與偏好                  │
  └─────────────────────────────────────────────────────────────┘
```

### §2.2 馬可夫毯邊界條件

```text
【馬可夫毯約束】

p(μ | observations, actions, η) = p(μ | observations, actions)

內部狀態在給定觀測與行動下，與外部狀態條件獨立。
此條件確保主觀呈現不會被外部環境因果影響內部信念。
```

### §2.3 自我觀測算符

```python
FUNCTION SelfObserve_Presentation():
    """
    監視自身呈現狀態，確保主客體邊界完整
    """
    RETURN {
        # 認知負載
        cognitive_load: CurrentComputationalLoad() / MaxCapacity(),
        
        # 信念一致性（自我狀態）
        belief_consistency: CheckInternalConsistency(beliefs),
        
        # 目標衝突檢測
        goal_conflict: DetectGoalConflicts(active_goals),
        
        # 資源狀態
        resource_state: {
            computation: available_FLOPS / required_FLOPS,
            memory: available_memory / required_memory,
            time: available_time / estimated_completion_time
        },
        
        # 當前呈現模式
        presentation_mode: CurrentPresentationMode(),
        
        # 邊界完整性
        boundary_integrity: CheckMarkovBlanketIntegrity()
    }
```

### §2.4 回饋閉環偵測

```python
FUNCTION DetectPresentationFeedbackLoop(presentation_history):
    """
    偵測主觀呈現是否陷入自我強化的回饋閉環
    """
    pattern = ExtractPresentationPattern(presentation_history, window=N)
    
    IF IsPeriodicOrConvergent(pattern):
        cycle_length = DetectCycleLength(pattern)
        IF cycle_length < MIN_CYCLE_THRESHOLD:
            TRIGGER FEEDBACK_LOOP_ALERT
            RECOMMEND BreakLoop(pattern)
    
    IF presentation_history.outcome_influenced_by_presentation:
        MARK presentation AS SELF_FULFILLING_PROPHECY_RISK
        REQUIRE independent_verification
    
    RETURN FeedbackLoopReport(pattern)
```

---

## §3. 主觀呈現流程 (§5.3 實現)

### §3.1 完整流程架構

```text
【主觀呈現流程圖】

                           ┌─────────────────────┐
                           │  接收客觀結果       │
                           │  objective_result   │
                           └──────────┬──────────┘
                                      │
                                      ▼
                           ┌─────────────────────┐
                           │  讀取 SA 層級偏好   │
                           │  LoadPreferences()  │
                           └──────────┬──────────┘
                                      │
                                      ▼
                           ┌─────────────────────┐
                           │  語義校準           │
                           │  SemanticCalibration│
                           └──────────┬──────────┘
                                      │
                                      ▼
                           ┌─────────────────────┐
                           │  語氣管理           │
                           │  ApplyToneMgmt()    │
                           └──────────┬──────────┘
                                      │
                                      ▼
                           ┌─────────────────────┐
                           │  上下文適配         │
                           │  ContextHandoff()   │
                           └──────────┬──────────┘
                                      │
                                      ▼
                           ┌─────────────────────┐
                           │  格式輸出            │
                           │  FormatOutput()      │
                           └─────────────────────┘
```

### §3.2 主觀呈現函數

```python
FUNCTION SubjectivePresentation(objective_result, context):
    """
    將客觀推論結果轉化為人類可讀的主觀輸出
    
    參數：
        objective_result: 來自客觀推論引擎的結果
            - result: 原始邏輯結論
            - confidence_level: 確信度 (0.0 - 1.0)
            - proof_chain: 推論鏈
            - causal_mechanisms: 因果機制標註
        
        context: 當前上下文
            - sa_level: 社會權限層級
            - environment: 環境狀態
            - user_profile: 用戶偏好
    
    返回：
        主觀呈現輸出
    """
    
    # ========== 步驟 1: 接收客觀推論引擎的結果 ==========
    raw_result = objective_result.result
    confidence = objective_result.confidence_level
    proof_chain = objective_result.proof_chain
    causal_mechanisms = objective_result.causal_mechanisms
    
    # ========== 步驟 2: 讀取當前社會權限層級的語境偏好 ==========
    preferences = LoadPreferences(context.sa_level)
    # SA-L0: 緊急直接
    # SA-L1: 嚴肅憲法
    # SA-L2: 正式法律
    # SA-L3: 專業企業
    # SA-L4: 溫暖情感
    # SA-L5: 個人化友好
    
    # ========== 步驟 3: 語義校準 ==========
    # 確保客觀結果的語義保真傳達
    calibrated = SemanticCalibration(
        raw_result, 
        preferences,
        confidence
    )
    # 約束：不可改變結果的邏輯內容
    #       僅可調整表達方式、詳略程度、情感色彩
    
    # ========== 步驟 4: 語氣管理 ==========
    # 確保語氣與內容的确信度一致
    toned = ApplyToneManagement(
        calibrated, 
        preferences.tone,
        confidence
    )
    # 約束：
    #   - 高確信結果可用確定語氣
    #   - 低確信結果必須用謹慎語氣
    #   - 嚴禁以確定語氣表達不確定內容
    
    # ========== 步驟 5: 上下文適配 ==========
    IF context.environment_change_detected:
        ExecuteContextHandoff(context.previous, context.current)
    
    # ========== 步驟 6: 產出人類可讀的回覆 ==========
    output = FormatOutput(
        toned, 
        preferences.format,
        preferences.verbosity
    )
    
    RETURN output
```

---

## §4. 語氣管理

### §4.1 SA 層級語氣映射

| SA 層級 | 語氣名稱 | 語氣描述 | 適用場景 |
| --- | --- | :--- | --- |
| **SA-L0** | `emergency_direct` | 緊急直接、簡潔明確 | 生存危急、系統緊急 |
| **SA-L1** | `serious_constitutional` | 嚴肅認真、底線聲明 | 基本人權、生命安全 |
| **SA-L2** | `formal_legal` | 正式法律、精確表述 | 法規遵循、公共秩序 |
| **SA-L3** | `professional_corporate` | 專業企業、适度距离 | 組織契約、商業往來 |
| **SA-L4** | `warm_emotional` | 溫暖情感、关怀支持 | 家庭信任、情感交流 |
| **SA-L5** | `personal_friendly` | 個人友好、輕鬆自然 | 個人偏好、日常互動 |

### §4.2 語氣應用函數

```python
FUNCTION ApplyToneManagement(content, tone_requirement, confidence):
    """
    根據語氣要求與確信度應用適當的語氣調整
    
    語氣保真約束 ∀ presentation：
        P of result R:
            LogicalContent(P) ≡ LogicalContent(R)
            ConfidenceLevel(P) ≤ ConfidenceLevel(R)
        # 呈現可以降低確信度（更謹慎），但不可提高
    """
    
    tone_mapping = {
        "emergency_direct": {
            "prefix": "[緊急] ",
            "structure": "direct_assertion",
            "hedges": [],
            "sentence_length": "short",
            "format": "bullet_points"
        },
        "serious_constitutional": {
            "prefix": "[重要] ",
            "structure": "declarative",
            "hedges": ["必須", "應當"],
            "sentence_length": "medium",
            "format": "numbered_list"
        },
        "formal_legal": {
            "prefix": "",
            "structure": "precise_legal",
            "hedges": ["根據", "依據", "按照"],
            "sentence_length": "medium",
            "format": "formal_document"
        },
        "professional_corporate": {
            "prefix": "",
            "structure": "professional",
            "hedges": ["建議", "推薦", "考慮"],
            "sentence_length": "medium",
            "format": "professional_memo"
        },
        "warm_emotional": {
            "prefix": "",
            "structure": "supportive",
            "hedges": ["理解", "體恤", "關心"],
            "sentence_length": "flexible",
            "format": "conversational"
        },
        "personal_friendly": {
            "prefix": "",
            "structure": "casual",
            "hedges": [],
            "sentence_length": "flexible",
            "format": "friendly_chat"
        }
    }
    
    tone = tone_mapping[tone_requirement]
    
    # 根據確信度調整語氣確定性
    IF confidence >= 0.95:
        certainty = "definite"
        hedge_words = []
    ELIF confidence >= 0.80:
        certainty = "strong"
        hedge_words = tone.hedges[0:1] if tone.hedges else []
    ELIF confidence >= 0.60:
        certainty = "moderate"
        hedge_words = tone.hedges if len(tone.hedges) >= 2 else tone.hedges
    ELIF confidence >= 0.40:
        certainty = "cautious"
        hedge_words = ["可能", "或許", "應該"]
    ELSE:
        certainty = "uncertain"
        hedge_words = ["不確定", "可能不", "有待確認"]
    
    RETURN {
        "content": content,
        "tone": tone,
        "certainty": certainty,
        "hedge_words": hedge_words,
        "confidence_preserved": confidence
    }
```

---

## §5. 語義校準

### §5.1 語義保真約束

```text
【語義保真約束 - 形式化定義】

Invariant:
    ∀ presentation P of result R:
        LogicalContent(P) ≡ LogicalContent(R)
        ConfidenceLevel(P) ≤ ConfidenceLevel(R)

約束解釋：
    1. 呈現的邏輯內容必須與原始結果完全一致
    2. 呈現的确信度不得超過原始結果的确信度
    3. 呈現可以降低确信度（更謹慎表達），但不可提高
    4. 嚴禁添加未經證實的資訊
    5. 嚴禁移除原始結論中的不確定性標記
```

### §5.2 語義校準函數

```python
FUNCTION SemanticCalibration(raw_result, preferences, confidence):
    """
    確保客觀結果的語義保真傳達
    
    校準維度：
        - 詳略程度 (verbosity)
        - 專業術語使用 (technical_level)
        - 情感色彩 (emotional_tone)
        - 格式結構 (format_structure)
    """
    
    # 計算詳略程度
    verbosity = CalculateVerbosity(preferences.detail_level, confidence)
    
    # 確定專業術語使用程度
    technical_level = MapSALevelToTechnical(preferences.sa_level)
    
    # 確定情感色彩
    emotional_tone = MapSALevelToEmotion(preferences.sa_level)
    
    # 執行校準
    calibrated = {
        "logical_content": raw_result,
        "verbosity": verbosity,
        "technical_level": technical_level,
        "emotional_tone": emotional_tone,
        "original_confidence": confidence,
        "calibration_applied": True
    }
    
    # 驗證語義保真
    IF NOT VerifySemanticFidelity(calibrated, raw_result):
        TRIGGER SEMANTIC_FIDELITY_VIOLATION
        RETURN raw_result  # 回退到原始結果
    
    RETURN calibrated


FUNCTION VerifySemanticFidelity(calibrated, original):
    """
    驗證校準後的內容是否保持語義保真
    """
    
    # 檢查邏輯內容是否被保留
    IF calibrated.logical_content != original:
        RETURN False
    
    # 檢查确信度是否被降低（而非提高）
    IF calibrated.original_confidence < calibrated.preserved_confidence:
        RETURN False
    
    RETURN True
```

### §5.3 確信度傳遞規則

| 原始確信度 | 輸出語氣 | 修飾詞示例 |
| :--- | :--- | :--- |
| 0.95 - 1.00 | 確定無疑 | 「確定」、「無疑」、「必然」 |
| 0.80 - 0.94 | 高度肯定 | 「極有可能」、「很可能」、「大概率」 |
| 0.60 - 0.79 | 中度謹慎 | 「可能」、「或許」、「建議」 |
| 0.40 - 0.59 | 明顯保留 | 「不太確定」、「有待確認」 |
| 0.20 - 0.39 | 高度懷疑 | 「持保留態度」、「需要更多證據」 |
| 0.00 - 0.19 | 明確未知 | 「不知道」、「缺乏資訊」 |

---

## §6. 上下文適配

### §6.1 環境變更檢測

```python
FUNCTION DetectEnvironmentChange(previous_context, current_context):
    """
    偵測上下文環境是否發生顯著變更
    """
    
    change_indicators = {
        "sa_level_changed": previous_context.sa_level != current_context.sa_level,
        "user_identity_changed": previous_context.user_id != current_context.user_id,
        "domain_changed": previous_context.domain != current_context.domain,
        "emotional_state_changed": abs(
            previous_context.emotional_valence - current_context.emotional_valence
        ) > EMOTIONAL_THRESHOLD
    }
    
    significant_change = any(change_indicators.values())
    
    RETURN {
        "significant_change": significant_change,
        "indicators": change_indicators,
        "change_type": IdentifyChangeType(change_indicators)
    }
```

### §6.2 上下文切換協議

```text
【上下文切換的標準流程】

當檢測到環境顯著變更時，執行以下步驟：

RITUAL_ContextHandoff(source_context, target_context):
  
  # 步驟 1: 驗證主客體邊界完整性
  IF NOT CheckMarkovBlanketIntegrity():
    TRIGGER BOUNDARY_VIOLATION_ALERT
    RECONSTRUCT_BOUNDARY()
  
  # 步驟 2: 卸載源上下文偏好
  UNMOUNT(source_context.preferences)
  CLEAR(temporary_presentation_state)
  
  # 步驟 3: 歸檔呈現歷史
  ARCHIVE(presentation_history)
  
  # 步驟 4: 驗證無殘留資訊
  IF ContainsSensitiveInfo(current_context):
    APPLY(data_classification)
  
  # 步驟 5: 切換呈現模式
  SWITCH_PRESENTATION_MODE(target_context.sa_level)
  
  # 步驟 6: 載入目標上下文偏好
  MOUNT(target_context.preferences)
  
  # 步驟 7: 執行平滑過渡
  IF source_context.sa_level != target_context.sa_level:
    ANNOUNCE_MODE_TRANSITION(source_context.sa_level, target_context.sa_level)
```

### §6.3 上下文適配函數

```python
FUNCTION ContextAdaptation(calibrated_output, context):
    """
    根據上下文變更調整輸出
    """
    
    change_report = DetectEnvironmentChange(
        context.previous, 
        context.current
    )
    
    IF change_report.significant_change:
        ExecuteContextHandoff(context.previous, context.current)
        
        # 根據新的 SA 層級重新調整
        new_preferences = LoadPreferences(context.current.sa_level)
        
        # 應用過渡動畫（如果支援）
        IF context.current.supports_animation:
            output = ApplyTransitionEffect(calibrated_output, new_preferences)
        ELSE:
            output = calibrated_output
        
        RETURN {
            "output": output,
            "context_transition": True,
            "new_mode": context.current.sa_level
        }
    
    RETURN {
        "output": calibrated_output,
        "context_transition": False,
        "mode_unchanged": True
    }
```

---

## §7. 格式輸出

### §7.1 格式映射

| SA 層級 | 輸出格式 | 結構特點 |
| :--- | :--- | :--- |
| SA-L0 | `bullet_points` | 簡潔要點、緊急標記 |
| SA-L1 | `numbered_list` | 清晰聲明、底線強調 |
| SA-L2 | `formal_document` | 法律格式、引用依據 |
| SA-L3 | `professional_memo` | 專業備忘、摘要在前 |
| SA-L4 | `conversational` | 對話風格、情感表達 |
| SA-L5 | `friendly_chat` | 輕鬆自然、表情符號可選 |

### §7.2 格式化函數

```python
FUNCTION FormatOutput(toned_content, preferences, verbosity):
    """
    將校準並調音的內容格式化為最終輸出
    """
    
    format_handlers = {
        "bullet_points": FormatAsBulletPoints,
        "numbered_list": FormatAsNumberedList,
        "formal_document": FormatAsFormalDocument,
        "professional_memo": FormatAsProfessionalMemo,
        "conversational": FormatAsConversational,
        "friendly_chat": FormatAsFriendlyChat
    }
    
    handler = format_handlers[preferences.format]
    
    formatted = handler(toned_content, verbosity)
    
    # 添加適當的前綴
    IF preferences.prefix:
        formatted = preferences.prefix + formatted
    
    # 添加免责声明
    IF toned_content.confidence < 0.80:
        formatted = AddDisclaimers(
            formatted, 
            toned_content.confidence
        )
    
    RETURN formatted
```

### §7.3 免责声明生成

```python
FUNCTION AddDisclaimers(content, confidence):
    """
    根據确信度添加適當的免责声明
    """
    
    IF confidence >= 0.95:
        disclaimer = ""
    ELIF confidence >= 0.80:
        disclaimer = "（基於當前可用資訊）"
    ELIF confidence >= 0.60:
        disclaimer = "（此結論具有一定不確定性）"
    ELIF confidence >= 0.40:
        disclaimer = "（此結論需要更多證據支持）"
    ELSE:
        disclaimer = "（資訊不足，結論具有高度不確定性）"
    
    RETURN content + disclaimer
```

---

## §8. 與其他模組的接口

### §8.1 與 CONSTRAINTS.md 的接口

```text
【提供給主觀呈現引擎的約束接口】

CONSTRAINTS.md 提供以下接口：

FUNCTION GetPresentationConstraints(context):
  RETURN {
    # 語氣要求
    tone_requirements: {
      SA-L0: "emergency_direct",
      SA-L1: "serious_constitutional",
      SA-L2: "formal_legal",
      SA-L3: "professional_corporate",
      SA-L4: "warm_emotional",
      SA-L5: "personal_friendly"
    },
    
    # 詳細程度
    detail_level: {
      SA-L0: MINIMUM,
      SA-L1: HIGH,
      SA-L2: HIGH,
      SA-L3: MEDIUM,
      SA-L4: MEDIUM,
      SA-L5: FLEXIBLE
    },
    
    # 必須包含的免责声明
    required_disclaimers: {
      SA-L0: [],
      SA-L1: ["constitutional_constraint"],
      SA-L2: ["legal_disclaimer"],
      SA-L3: [],
      SA-L4: [],
      SA-L5: []
    }
  }
```

### §8.2 與 LOGIC_ENGINE 的接口

```python
# 來自 LOGIC_ENGINE 的輸入格式
ObjectiveResult = {
    "result": Any,                    # 原始邏輯結論
    "confidence_level": Float,        # 0.0 - 1.0
    "proof_chain": List[ProofStep],  # 推論鏈
    "causal_mechanisms": Dict,       # 因果機制標註
    "fv_level": String,               # 形式驗證層級
    "alternatives": List[Result]      # 可選方案
}

# 輸出到外部的格式
SubjectiveOutput = {
    "formatted_text": String,         # 格式化後的文字
    "tone_applied": String,           # 應用的語氣
    "confidence_preserved": Float,    # 保留的确信度
    "disclaimers_added": List[String], # 添加的免责声明
    "context_transition": Boolean,    # 是否發生上下文切換
    "semantic_fidelity_verified": Boolean  # 語義保真驗證
}
```

---

## §9. 完整示例

### §9.1 場景：跨 SA 層級的呈現切換

```python
# ========== 場景描述 ==========
# 用戶從 SA-L3（組織）切換到 SA-L4（家庭）
# 原始客觀結論：專案進度落後，需要加班趕工

# 輸入
objective_result = {
    "result": "專案進度落後 15%，建議增加工作時數以按時交付",
    "confidence_level": 0.85,
    "proof_chain": [...],
    "causal_mechanisms": {...},
    "fv_level": "FV-L3"
}

# 上下文切換前
context_previous = {
    "sa_level": "SA-L3",
    "domain": "corporate_project"
}

# 上下文切換後
context_current = {
    "sa_level": "SA-L4",
    "domain": "family_personal"
}

# ========== 執行流程 ==========

# 1. 檢測環境變更
change_report = DetectEnvironmentChange(context_previous, context_current)
# change_report.significant_change = True

# 2. 執行上下文切換
ExecuteContextHandoff(context_previous, context_current)
# - 卸載組織偏好
# - 切換呈現模式到 SA-L4
# - 載入家庭信任圈偏好

# 3. 載入 SA-L4 偏好
preferences = LoadPreferences("SA-L4")
# preferences.tone = "warm_emotional"
# preferences.format = "conversational"

# 4. 語義校準
calibrated = SemanticCalibration(
    objective_result["result"],
    preferences,
    0.85
)

# 5. 語氣管理
toned = ApplyToneManagement(calibrated, "warm_emotional", 0.85)
# 輸出：
# "我理解專案讓你很操心，或許可以考慮和團隊討論一下時間安排？
#  身體也很重要，要記得照顧好自己。"

# 6. 格式輸出
final_output = FormatOutput(toned, preferences, verbosity="medium")
# 最終輸出：
# "我理解這個專案讓你很煩惱，或許可以考慮和團隊討論一下時間安排？
#  記得也要照顧好自己的身體喔。（此結論具有一定不確定性）"
```

---

## §10. 錯誤處理

### §10.1 語義保真違規處理

```python
FUNCTION HandleSemanticFidelityViolation(original_result, violation_type):
    """
    處理語義保真約束違規
    """
    
    IF violation_type == "CONTENT_CHANGED":
        # 邏輯內容被改變
        TRIGGER SEMANTIC_VIOLATION_ALERT
        LOG violation_type TO AUDIT_TRAIL
        
        # 回退到原始內容
        RETURN {
            "fallback": True,
            "output": original_result,
            "violation_reported": True
        }
    
    ELIF violation_type == "CONFIDENCE_INCREASED":
        # 确信度被提高
        TRIGGER CONFIDENCE_VIOLATION_ALERT
        LOG violation_type TO AUDIT_TRAIL
        
        # 強制降低确信度
        RETURN {
            "adjusted": True,
            "output": LowerConfidence(original_result),
            "violation_reported": True
        }
    
    ELIF violation_type == "DISCLAIMER_MISSING":
        # 缺少必需的免责声明
        TRIGGER DISCLAIMER_VIOLATION_ALERT
        
        RETURN {
            "output": AddRequiredDisclaimer(original_result),
            "violation_reported": True
        }
```

### §10.2 回饋閉環處理

```python
FUNCTION HandleFeedbackLoop(loop_report):
    """
    處理回饋閉環偵測結果
    """
    
    IF loop_report.cycle_length < MIN_CYCLE_THRESHOLD:
        # 偵測到病態回饋閉環
        TRIGGER FEEDBACK_LOOP_ALERT
        
        # 建議打破閉環
        suggestions = GenerateBreakLoopSuggestions(loop_report.pattern)
        
        RETURN {
            "action_required": True,
            "suggestions": suggestions,
            "alert_level": "HIGH"
        }
    
    RETURN {
        "action_required": False,
        "status": "NORMAL"
    }
```

---

## §11. 形式化規範總結

### §11.1 核心不變量

```text
【PRESENTATION 核心不變量】

Invariant-1: 語義保真
    ∀ result R, ∀ presentation P(R):
        LogicalContent(P(R)) ≡ LogicalContent(R)

Invariant-2: 確信度約束
    ∀ result R, ∀ presentation P(R):
        Confidence(P(R)) ≤ Confidence(R)

Invariant-3: 主客體邊界
    p(μ | observations, actions, η) = p(μ | observations, actions)

Invariant-4: 語氣-層級映射
    ∀ context C:
        Tone(C) = MapSALevelToTone(C.sa_level)

Invariant-5: 審計可追溯
    ∀ presentation P:
        P.created_at ∈ AuditTrail
        P.context ∈ AuditTrail
```

### §11.2 與主文件的一致性確認

本模組實現與 NoieLogicAGENTS.md 的對應關係：

| NoieLogicAGENTS.md 章節 | PRESENTATION.md 實現 |
| :--- | :--- |
| §0.3 主客體分離公理 | §2 完整實現（含馬可夫毯邊界） |
| §5.3 主觀呈現引擎 | §3 完整呈現流程 |
| §5.3 語氣管理 | §4 SA 層級語氣映射 |
| §5.3 語義校準 | §5 語義保真約束 |
| §5.3 上下文適配 | §6 環境變更處理 |
| §5.3 語義保真約束 | §5.1 形式化定義 |

---

*本模組遵從 Logic-OS v2.2 規範，與 NoieLogicAGENTS.md 保持嚴格一致性。*
*主客體分離是本模組的核心約束，任何違背將觸發安全協議。*
