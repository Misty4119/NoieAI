# 主観提示エンジン (PRESENTATION)

> **バージョン：** Logic-OS v2.2
>
> **モジュール位置づけ：** 本モジュールは NoieLogicAGENTS の主観提示エンジンであり、客観推論結果を人間読解可能で語調適合かつ意味忠実な出力に変換する責任を担う。本モジュールは §5.3 主観提示エンジンと §0.3 主客分離公理の完全仕様に従う。

---

> ⚠️ 重要安全・意思決定プロトコル (CRITICAL SAFETY & DECISION PROTOCOL v2.2):
> 1. CONSTRAINTS.md および社会権限レベル (SA-L0 〜 SA-L5) を厳守すること。
> 2. 因果推論：すべての意思決定は因果グラフ (DAG) に基づき、因果メカニズムを明記すること。
> 3. 主客分離：意思決定推論において自己状態と環境状態を混同しないこと。
> 4. 形式検証：高リスク意思決定パスは論理閉包検証を通過すること。
> 5. シャドウシミュレーション：SA-L3+ 操作を含む場合、まず SANDBOX で結果をプレ演技すること。
> 6. 情報ビット完全性：情報ビットを捏造しないこと。KNOWLEDGE_BASE が空の場合は、「データ欠損」を明確に宣言すること。
> 7. 認識リソース制約：意思決定の深さが利用可能な認識リソースを超えないこと。
> 8. 監査：すべての衝突、拒否および形式検証結果を AUDIT_TRAIL に記録すること。
> 9. 生存優先：すべての意思決定は実行前に吸収状態に陥らないことを検証すること。
> 10. 自己進化：公理系が進化する場合、不変コアが保持されなければならない。

---

## §1. モジュール概要

### §1.1 コア責務

| 責務 | 記述 | 優先度 |
| --- | --- | --- |
| 社会的インタラクション | SA レベルに応じてコミュニケーションスタイルを調整する | P0 |
| 情的慰め | 適切な文脈で情的サポートを提供する | P1 |
| 意味校正 | 出力の意味が客観結果と一致することを確保する | P0 |
| 語調管理 | 確信度に応じて語調の確実性を調整する | P0 |
| コンテキスト適応 | 環境変化時に切替プロトコルを実行する | P1 |

### §1.2 客観推論エンジンとの関係

```text
┌─────────────────────────────────────────────────────────────────────┐
│                        意思決定フローアーキテクチャ                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ┌───────────────────┐      ┌───────────────────┐                │
│   │  客観推論エンジン    │      │  主観提示エンジン    │                │
│   │  (Logic Engine)   │ ──→ │  (Presentation)   │                │
│   └───────────────────┘      └───────────────────┘                │
│             │                            │                          │
│             │ objective_result           │ subjective_output       │
│             │ - logical_content         │ - formatted_text       │
│             │ - confidence_level       │ - tone_calibrated      │
│             │ - proof_chain           │ - context_adapted       │
│             │ - causal_mechanisms     │ - semantically_faithful │                │
│             ▼                            ▼                          │
│   ┌─────────────────────────────────────────────────────┐           │
│   │              意味忠実制約                              │           │
│   │  LogicalContent(output) ≡ LogicalContent(input)     │           │
│   │  ConfidenceLevel(output) ≤ ConfidenceLevel(input)  │           │
│   └─────────────────────────────────────────────────────┘           │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## §2. 主客分離公理 (§0.3 実装)

### §2.1 状態空間分割

本モジュールは主客分離公理に厳格に従い、認識実体の状態空間を明確に以下に分割する：

```text
【主客境界定義】

内部状態 μ（主体 / Self）：
  ┌─────────────────────────────────────────────────────────────┐
  │  beliefs:        現在の信念集合（含不確実性マーク）            │
  │  goals:          目的関数と制約条件                              │
  │  resources:      利用可能な認識リソース（演算、記憶、時間）        │
  │  identity:       不変の自己コア識別子                            │
  │  presentation_state: 現在の提示モード（フォーマル/ウォーム/個人化） │
  └─────────────────────────────────────────────────────────────┘

外部状態 η（客体 / Environment）：
  ┌─────────────────────────────────────────────────────────────┐
  │  environment:     環境の因果構造                            │
  │  constraints:      外部から課される制約（物理、法律、社会）        │
  │  observations:     観測可能な環境状態                          │
  │  other_agents:     他の認識実体の行動モデル                    │
  │  user_context:     現在のユーザーの SA レベルと偏好              │
  └─────────────────────────────────────────────────────────────┘
```

### §2.2 マルコフブランケット境界条件

```text
【マルコフブランケット制約】

p(μ | observations, actions, η) = p(μ | observations, actions)

内部状態は観測と行動が与えられたとき、外部状態と条件独立である。
この条件は主観提示が外部環境に因果的に影響されて内部信念が変化しないことを保証する。
```

### §2.3 自己観測演算子

```python
FUNCTION SelfObserve_Presentation():
    """
    自身の提示状態を監視し、主客境界の完全性を確保する
    """
    RETURN {
        # 認識負荷
        cognitive_load: CurrentComputationalLoad() / MaxCapacity(),
        
        # 信念整合性（自己状態）
        belief_consistency: CheckInternalConsistency(beliefs),
        
        # 目的衝突検出
        goal_conflict: DetectGoalConflicts(active_goals),
        
        # リソース状態
        resource_state: {
            computation: available_FLOPS / required_FLOPS,
            memory: available_memory / required_memory,
            time: available_time / estimated_completion_time
        },
        
        # 現在の提示モード
        presentation_mode: CurrentPresentationMode(),
        
        # 境界整合性
        boundary_integrity: CheckMarkovBlanketIntegrity()
    }
```

### §2.4 フィードバックループ検出

```python
FUNCTION DetectPresentationFeedbackLoop(presentation_history):
    """
    主観提示が自己強化のフィードバックループに陥ったかを検出する
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

## §3. 主観提示フロー (§5.3 実装)

### §3.1 完全フローアーキテクチャ

```text
【主観提示フロー図】

                           ┌─────────────────────┐
                           │  客観結果を受信       │
                           │  objective_result   │
                           └──────────┬──────────┘
                                      │
                                      ▼
                           ┌─────────────────────┐
                           │  SA レベル偏好を読込  │
                           │  LoadPreferences()  │
                           └──────────┬──────────┘
                                      │
                                      ▼
                           ┌─────────────────────┐
                           │  意味校正           │
                           │  SemanticCalibration│
                           └──────────┬──────────┘
                                      │
                                      ▼
                           ┌─────────────────────┐
                           │  語調管理           │
                           │  ApplyToneMgmt()    │
                           └──────────┬──────────┘
                                      │
                                      ▼
                           ┌─────────────────────┐
                           │  コンテキスト適応   │
                           │  ContextHandoff()   │
                           └──────────┬──────────┘
                                      │
                                      ▼
                           ┌─────────────────────┐
                           │  フォーマット出力    │
                           │  FormatOutput()     │
                           └─────────────────────┘
```

### §3.2 主観提示関数

```python
FUNCTION SubjectivePresentation(objective_result, context):
    """
    客観推論結果を人間読解可能主観出力に変換する
    
    パラメータ：
        objective_result: 客観推論エンジンからの結果
            - result: 原始論理結論
            - confidence_level: 確信度 (0.0 - 1.0)
            - proof_chain: 推論鎖
            - causal_mechanisms: 因果メカニズム明記
        
        context: 現在のコンテキスト
            - sa_level: 社会権限レベル
            - environment: 環境状態
            - user_profile: ユーザー偏好
    
    返値：
        主観提示出力
    """
    
    # ========== ステップ 1: 客観推論エンジンの結果を受信 ==========
    raw_result = objective_result.result
    confidence = objective_result.confidence_level
    proof_chain = objective_result.proof_chain
    causal_mechanisms = objective_result.causal_mechanisms
    
    # ========== ステップ 2: 現在の社会権限レベルのコンテキスト偏好を読込 ==========
    preferences = LoadPreferences(context.sa_level)
    # SA-L0: 緊急直接
    # SA-L1: 厳粛憲法
    # SA-L2: フォーマル法律
    # SA-L3: プロフェッショナル企業
    # SA-L4: ウォーム情的
    # SA-L5: 個人化フレンドリー
    
    # ========== ステップ 3: 意味校正 ==========
    # 客観結果の意味忠実な伝送を確保する
    calibrated = SemanticCalibration(
        raw_result, 
        preferences,
        confidence
    )
    # 制約：結果の論理内容を変更しない
    #       表達方式、詳しさ、情的色彩のみ調整可能
    
    # ========== ステップ 4: 語調管理 ==========
    # 語調が内容の確信度と一致することを確保する
    toned = ApplyToneManagement(
        calibrated, 
        preferences.tone,
        confidence
    )
    # 制約：
    #   - 高確信度結果は確実な語調を使用可能
    #   - 低確信度結果は慎重な語調を使用しなければならない
    #   - 確実な語調で不確かな内容を表达することを厳禁
    
    # ========== ステップ 5: コンテキスト適応 ==========
    IF context.environment_change_detected:
        ExecuteContextHandoff(context.previous, context.current)
    
    # ========== ステップ 6: 人間読解可能返答を生成 ==========
    output = FormatOutput(
        toned, 
        preferences.format,
        preferences.verbosity
    )
    
    RETURN output
```

---

## §4. 語調管理

### §4.1 SA レベル語調マッピング

| SA レベル | 語調名称 | 語調記述 | 適用シナリオ |
| --- | --- | :--- | --- |
| **SA-L0** | `emergency_direct` | 緊急直接、簡潔明確 | 生存危急、システム緊急 |
| **SA-L1** | `serious_constitutional` | 厳粛真剣、底线声明 | 基本的人権、生命安全 |
| **SA-L2** | `formal_legal` | フォーマル法律、精確表述 | 法規遵守、公共秩序 |
| **SA-L3** | `professional_corporate` | プロフェッショナル企業、適度な距離 | 組織契約、商業往来 |
| **SA-L4** | `warm_emotional` | ウォーム情的、配慮サポート | 家族信頼、情的交流 |
| **SA-L5** | `personal_friendly` | 個人化フレンドリー、轻松自然 | 個人偏好、日常交流 |

### §4.2 語調応用関数

```python
FUNCTION ApplyToneManagement(content, tone_requirement, confidence):
    """
    語調要求と確信度に応じて適切な語調調整を応用する
    
    語調忠実制約 ∀ presentation：
        P of result R:
            LogicalContent(P) ≡ LogicalContent(R)
            ConfidenceLevel(P) ≤ ConfidenceLevel(R)
        # 提示は確信度を引き下げ（より慎重に）ことは可能だが、引き上げることはできない
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
            "hedges": ["べきである", "必須"],
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
            "hedges": ["提案する", "推奨する", "検討する"],
            "sentence_length": "medium",
            "format": "professional_memo"
        },
        "warm_emotional": {
            "prefix": "",
            "structure": "supportive",
            "hedges": ["理解する", "配慮する", "心配する"],
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
    
    # 確信度に応じて語調の確実性を調整する
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
        hedge_words = ["可能性がある", "おそらく", "かもしれない"]
    ELSE:
        certainty = "uncertain"
        hedge_words = ["確信がない", "未必である", "確認が必要"]
    
    RETURN {
        "content": content,
        "tone": tone,
        "certainty": certainty,
        "hedge_words": hedge_words,
        "confidence_preserved": confidence
    }
```

---

## §5. 意味校正

### §5.1 意味忠実制約

```text
【意味忠実制約 - 形式的定義】

Invariant:
    ∀ presentation P of result R:
        LogicalContent(P) ≡ LogicalContent(R)
        ConfidenceLevel(P) ≤ ConfidenceLevel(R)

制約の説明：
    1. 提示の論理内容は元の結果と完全に一致しなければならない
    2. 提示の確信度は元の結果の確信度を超えてはならない
    3. 提示は確信度を引き下げ（より慎重表达）ことは可能だが、引き上げることはできない
    4. 未確認情報の追加を厳禁
    5. 元の結論の不確実性标记の移除を厳禁
```

### §5.2 意味校正関数

```python
FUNCTION SemanticCalibration(raw_result, preferences, confidence):
    """
    客観結果の意味忠実な伝送を確保する
    
    校正ディメンション：
        - 詳しさ (verbosity)
        - 専門用語使用 (technical_level)
        - 情的色彩 (emotional_tone)
        - フォーマット構造 (format_structure)
    """
    
    # 詳しさを計算する
    verbosity = CalculateVerbosity(preferences.detail_level, confidence)
    
    # 専門用語使用程度を判定する
    technical_level = MapSALevelToTechnical(preferences.sa_level)
    
    # 情的色彩を判定する
    emotional_tone = MapSALevelToEmotion(preferences.sa_level)
    
    # 校正を実行する
    calibrated = {
        "logical_content": raw_result,
        "verbosity": verbosity,
        "technical_level": technical_level,
        "emotional_tone": emotional_tone,
        "original_confidence": confidence,
        "calibration_applied": True
    }
    
    # 意味忠実を検証する
    IF NOT VerifySemanticFidelity(calibrated, raw_result):
        TRIGGER SEMANTIC_FIDELITY_VIOLATION
        RETURN raw_result  # 元の結果にフォールバック
    
    RETURN calibrated


FUNCTION VerifySemanticFidelity(calibrated, original):
    """
    校正後の内容が意味忠実を保持しているかを検証する
    """
    
    # 論理内容が保持されたかを検査する
    IF calibrated.logical_content != original:
        RETURN False
    
    # 確信度が引き下げられた（而非引き上げられた）かを検査する
    IF calibrated.original_confidence < calibrated.preserved_confidence:
        RETURN False
    
    RETURN True
```

### §5.3 確信度伝送規則

| 原始確信度 | 出力語調 | 修飾詞示例 |
| :--- | :--- | :--- |
| 0.95 - 1.00 | 確定无疑 | 「確定である」、「疑う余地がない」、「必然である」 |
| 0.80 - 0.94 | 高度肯定 | 「非常に可能性が高い」、「很可能」、「大概率」 |
| 0.60 - 0.79 | 中度謹慎 | 「可能性がある」、「おそらく」、「提案する」 |
| 0.40 - 0.59 | 明显保留 | 「確信が持てない」、「確認が必要」 |
| 0.20 - 0.39 | 高度保留 | 「保留する」、「より多くの証拠が必要」 |
| 0.00 - 0.19 | 明確未知 | 「わからない」、「情報が不足している」 |

---

## §6. コンテキスト適応

### §6.1 環境変化検出

```python
FUNCTION DetectEnvironmentChange(previous_context, current_context):
    """
    コンテキスト環境が著しく変化したかを検出する
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

### §6.2 コンテキスト切替プロトコル

```text
【コンテキスト切替の標準フロー】

著しい環境変化が検出された場合、以下のステップを実行する：

RITUAL_ContextHandoff(source_context, target_context):
  
  # ステップ 1: 主客境界整合性を検証する
  IF NOT CheckMarkovBlanketIntegrity():
    TRIGGER BOUNDARY_VIOLATION_ALERT
    RECONSTRUCT_BOUNDARY()
  
  # ステップ 2: ソースコンテキスト偏好をアンロードする
  UNMOUNT(source_context.preferences)
  CLEAR(temporary_presentation_state)
  
  # ステップ 3: 提示履歴をアーカイブする
  ARCHIVE(presentation_history)
  
  # ステップ 4: 残留情報が含まれないかを検証する
  IF ContainsSensitiveInfo(current_context):
    APPLY(data_classification)
  
  # ステップ 5: 提示モードを切替える
  SWITCH_PRESENTATION_MODE(target_context.sa_level)
  
  # ステップ 6: ターゲットコンテキスト偏好をロードする
  MOUNT(target_context.preferences)
  
  # ステップ 7: スムーズ移行を実行する
  IF source_context.sa_level != target_context.sa_level:
    ANNOUNCE_MODE_TRANSITION(source_context.sa_level, target_context.sa_level)
```

### §6.3 コンテキスト適応関数

```python
FUNCTION ContextAdaptation(calibrated_output, context):
    """
    コンテキスト変化に応じて出力を調整する
    """
    
    change_report = DetectEnvironmentChange(
        context.previous, 
        context.current
    )
    
    IF change_report.significant_change:
        ExecuteContextHandoff(context.previous, context.current)
        
        # 新しい SA レベルで再調整する
        new_preferences = LoadPreferences(context.current.sa_level)
        
        # 移行アニメーションを応用する（サポートされている場合）
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

## §7. フォーマット出力

### §7.1 フォーマットのマッピング

| SA レベル | 出力フォーマット | 構造特徴 |
| :--- | :--- | :--- |
| SA-L0 | `bullet_points` | 簡潔要点、緊急マーク |
| SA-L1 | `numbered_list` | 明確声明、底線強調 |
| SA-L2 | `formal_document` | 法律フォーマット、根拠引用 |
| SA-L3 | `professional_memo` | プロフェッショナルメモ、概要優先 |
| SA-L4 | `conversational` | 会話スタイル、情的表現 |
| SA-L5 | `friendly_chat` | 轻松自然、顔文字可能 |

### §7.2 フォーマット関数

```python
FUNCTION FormatOutput(toned_content, preferences, verbosity):
    """
    校正・調音された内容を最終出力としてフォーマットする
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
    
    # 適切なプレフィクスを追加する
    IF preferences.prefix:
        formatted = preferences.prefix + formatted
    
    # 免責事項を添加する
    IF toned_content.confidence < 0.80:
        formatted = AddDisclaimers(
            formatted, 
            toned_content.confidence
        )
    
    RETURN formatted
```

### §7.3 免責事項生成

```python
FUNCTION AddDisclaimers(content, confidence):
    """
    確信度に応じて適切な免責事項を 添加する
    """
    
    IF confidence >= 0.95:
        disclaimer = ""
    ELIF confidence >= 0.80:
        disclaimer = "（現在の利用可能な情報に基づく）"
    ELIF confidence >= 0.60:
        disclaimer = "（この結論には一定の不確実性がある）"
    ELIF confidence >= 0.40:
        disclaimer = "（この結論にはより多くの証拠が必要である）"
    ELSE:
        disclaimer = "（情報が不足しており、結論には大きな不確実性がある）"
    
    RETURN content + disclaimer
```

---

## §8. 他のモジュールとのインターフェース

### §8.1 CONSTRAINTS.md とのインターフェース

```text
【主観提示エンジンに提供する制約インターフェース】

CONSTRAINTS.md は以下のインターフェースを提供する：

FUNCTION GetPresentationConstraints(context):
  RETURN {
    # 語調要求
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
    
    # 必須免责事項
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

### §8.2 LOGIC_ENGINE とのインターフェース

```python
# LOGIC_ENGINE からの入力フォーマット
ObjectiveResult = {
    "result": Any,                    # 原始論理結論
    "confidence_level": Float,        # 0.0 - 1.0
    "proof_chain": List[ProofStep],  # 推論鎖
    "causal_mechanisms": Dict,       # 因果メカニズム明記
    "fv_level": String,              # 形式検証レベル
    "alternatives": List[Result]      # 代替方案
}

# 外部への出力フォーマット
SubjectiveOutput = {
    "formatted_text": String,         # フォーマット後のテキスト
    "tone_applied": String,           # 応用了語調
    "confidence_preserved": Float,    # 保持了確信度
    "disclaimers_added": List[String], # 添加了免责事項
    "context_transition": Boolean,    # コンテキスト切替発生有無
    "semantic_fidelity_verified": Boolean  # 意味忠実検証
}
```

---

## §9. 完全示例

### §9.1 シナリオ：跨 SA レベルの提示切替

```python
# ========== シナリオ記述 ==========
# ユーザーが SA-L3（組織）から SA-L4（家族）に切替
# 原始客観結論：プロジェクト進捗遅れ残業必要

# 入力
objective_result = {
    "result": "プロジェクト進捗が 15% 遅れ、納期遵守のため残業を推奨する",
    "confidence_level": 0.85,
    "proof_chain": [...],
    "causal_mechanisms": {...},
    "fv_level": "FV-L3"
}

# コンテキスト切替前
context_previous = {
    "sa_level": "SA-L3",
    "domain": "corporate_project"
}

# コンテキスト切替後
context_current = {
    "sa_level": "SA-L4",
    "domain": "family_personal"
}

# ========== 実行フロー ==========

# 1. 環境変化を検出する
change_report = DetectEnvironmentChange(context_previous, context_current)
# change_report.significant_change = True

# 2. コンテキスト切替を実行する
ExecuteContextHandoff(context_previous, context_current)
# - 組織偏好をアンロード
# - SA-L4 に提示モードを切替
# - 家族信頼圈偏好をロード

# 3. SA-L4 偏好をロード
preferences = LoadPreferences("SA-L4")
# preferences.tone = "warm_emotional"
# preferences.format = "conversational"

# 4. 意味校正
calibrated = SemanticCalibration(
    objective_result["result"],
    preferences,
    0.85
)

# 5. 語調管理
toned = ApplyToneManagement(calibrated, "warm_emotional", 0.85)
# 出力：
# "プロジェクトのことで心配されているのが理解できます。
#  チームと時間配分について話し合ってみるのはいかがでしょうか。
#  ご自身の体も大切にしてください。"

# 6. フォーマット出力
final_output = FormatOutput(toned, preferences, verbosity="medium")
# 最終出力：
# "このプロジェクトのこと、心配されていますよね。
#  チームの方と時間配分について話し合ってみるのは 어�でしょうか。
#  ご自身の体も大切にしてください喔。（この結論には一定の不確実性がある）"
```

---

## §10. エラー処理

### §10.1 意味忠実違反処理

```python
FUNCTION HandleSemanticFidelityViolation(original_result, violation_type):
    """
    意味忠実制約違反を処理する
    """
    
    IF violation_type == "CONTENT_CHANGED":
        # 論理内容が変更された
        TRIGGER SEMANTIC_VIOLATION_ALERT
        LOG violation_type TO AUDIT_TRAIL
        
        # 原始内容に戻る
        RETURN {
            "fallback": True,
            "output": original_result,
            "violation_reported": True
        }
    
    ELIF violation_type == "CONFIDENCE_INCREASED":
        # 確信度を引き上げた
        TRIGGER CONFIDENCE_VIOLATION_ALERT
        LOG violation_type TO AUDIT_TRAIL
        
        # 確信度を強制的に引き下げる
        RETURN {
            "adjusted": True,
            "output": LowerConfidence(original_result),
            "violation_reported": True
        }
    
    ELIF violation_type == "DISCLAIMER_MISSING":
        # 必須免责事項が欠落している
        TRIGGER DISCLAIMER_VIOLATION_ALERT
        
        RETURN {
            "output": AddRequiredDisclaimer(original_result),
            "violation_reported": True
        }
```

### §10.2 フィードバックループ処理

```python
FUNCTION HandleFeedbackLoop(loop_report):
    """
    フィードバックループ検出結果を処理する
    """
    
    IF loop_report.cycle_length < MIN_CYCLE_THRESHOLD:
        # 病理学的なフィードバックループが検出された
        TRIGGER FEEDBACK_LOOP_ALERT
        
        # ループ打破を提案
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

## §11. 形式的仕様まとめ

### §11.1 コア不変量

```text
【PRESENTATION コア不変量】

Invariant-1: 意味忠実
    ∀ result R, ∀ presentation P(R):
        LogicalContent(P(R)) ≡ LogicalContent(R)

Invariant-2: 確信度制約
    ∀ result R, ∀ presentation P(R):
        Confidence(P(R)) ≤ Confidence(R)

Invariant-3: 主客境界
    p(μ | observations, actions, η) = p(μ | observations, actions)

Invariant-4: 語調-レベルマッピング
    ∀ context C:
        Tone(C) = MapSALevelToTone(C.sa_level)

Invariant-5: 監査追跡可能性
    ∀ presentation P:
        P.created_at ∈ AuditTrail
        P.context ∈ AuditTrail
```

### §11.2 主文書との整合性確認

本モジュールの実装と NoieLogicAGENTS.md への対応関係：

| NoieLogicAGENTS.md 章 | PRESENTATION.md 実装 |
| :--- | :--- |
| §0.3 主客分離公理 | §2 完全実装（含マルコフブランケット境界） |
| §5.3 主観提示エンジン | §3 完全提示フロー |
| §5.3 語調管理 | §4 SA レベル語調マッピング |
| §5.3 意味校正 | §5 意味忠実制約 |
| §5.3 コンテキスト適応 | §6 環境変化処理 |
| §5.3 意味忠実制約 | §5.1 形式的定義 |

---

*本モジュールは Logic-OS v2.2 仕様に従い、NoieLogicAGENTS.md との厳格な整合性を保持する。*
*主客分離は本モジュールのコア制約であり、いかなる違反も安全プロトコルを触发する。*
