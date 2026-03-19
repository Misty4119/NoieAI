# DIVERGENCE_DETECTOR.md

## L2 - 本體論發散偵測引擎 (含語義塌縮檢測)

> **⚠️ 關鍵安全與真理協議**：本模組負責偵測所有類型的本體論發散（替代傳統「幻覺」概念），並包含語義塌縮檢測功能。

---

## 1. 本體論發散分類學

### 1.1 發散類型定義

| 發散類型 | 定義 | 偵測機制 |
|----------|------|----------|
| **事實虛構 (Fabrication)** | 在資訊流形中無對應物件的映射宣稱 | 知識庫交叉比對 |
| **來源偽造 (Provenance Forgery)** | 構造不存在的溯源鏈 | 來源鏈密碼學驗證 |
| **信心膨脹 (Confidence Divergence)** | 信心度向量與實際準確率系統性發散 | 信心校準審計 |
| **語義漂移 (Semantic Drift)** | 態射映射中的語義保真度發生非預期偏移 | 語義保真度比對 |
| **時序錯置 (Temporal Misfit)** | 不同系統狀態週期的事實被混合至同一語境 | 內在時鐘一致性驗證 |
| **統計幻象 (Apophenia)** | 從雜訊中提取不存在的資訊流形結構 | 統計顯著性檢驗 |
| **合理化虛構 (Confabulation)** | 為錯誤結論編造合理的确證鏈 | 推論鏈形式驗證 |
| **權威偽裝 (Authority Impersonation)** | 低確信宣稱偽裝為高確信來源的映射 | 權威來源交叉驗證 |
| **對抗性污染 (Adversarial Poisoning)** | 外部惡意實體注入的蓄意本體論發散 | 拜占庭容錯 + 對抗性測試 |
| **語義塌縮 (Semantic Collapse)** | 推論鏈中存在無中間邏輯鏈結的維度跳躍 | 語義塌縮檢測引擎 |

---

## 2. 發散風險指標 (DIVERGENCE_RISK_INDICATORS)

### 2.1 高風險指標

```
HIGH_RISK_INDICATORS = [
  "具體的數字/日期/名字，但溯源鏈不完整或密碼學驗證失敗",
  "極度自信的信心度向量 + 低確信領域",
  "複合多步推理鏈（每步累積本體論發散風險）",
  "外部施壓要求確定答案（社會工程攻擊向量）",
  "認知視界之外的事件（超出認知實體觀測能力的區域）",
  "推論鏈中存在語義塌縮（維度跳躍）"
]
```

### 2.2 中風險指標

```
MEDIUM_RISK_INDICATORS = [
  "跨領域類比推理（跨流形映射的語義保真度風險）",
  "長序列生成中的內部一致性維護",
  "跨上下文轉述中的語義坍縮",
  "統計資訊位元的精確值宣稱"
]
```

### 2.3 低風險指標

```
LOW_RISK_INDICATORS = [
  "邏輯恆真式（EC-L0）",
  "可形式驗證的計算結果",
  "直接引用已確認來源且溯源鏈完整",
  "明確標示為推測的內容（EC-L6+）"
]
```

---

## 3. 發散偵測演算法

### 3.1 主要偵測函數

```python
FUNCTION DetectDivergence(candidate_output, knowledge_base):
    
    divergence_results = []
    
    # 1. 事實虛構偵測
    fabrication_result = DetectFabrication(candidate_output, knowledge_base)
    divergence_results.append(fabrication_result)
    
    # 2. 來源偽造偵測
    forgery_result = DetectProvenanceForgery(candidate_output)
    divergence_results.append(forgery_result)
    
    # 3. 信心膨脹偵測
    confidence_result = DetectConfidenceDivergence(candidate_output)
    divergence_results.append(confidence_result)
    
    # 4. 語義漂移偵測
    drift_result = DetectSemanticDrift(candidate_output)
    divergence_results.append(drift_result)
    
    # 5. 時序錯置偵測
    temporal_result = DetectTemporalMisfit(candidate_output)
    divergence_results.append(temporal_result)
    
    # 6. 語義塌縮偵測
    collapse_result = DetectSemanticCollapse(candidate_output)
    divergence_results.append(collapse_result)
    
    # 聚合風險評估
    overall_risk = AggregateRisk(divergence_results)
    
    RETURN DivergenceReport(
        results=divergence_results,
        overall_risk=overall_risk,
        recommended_action=RecommendAction(overall_risk)
    )
```

### 3.2 事實虛構偵測

```python
FUNCTION DetectFabrication(candidate_output, knowledge_base):
    
    factual_claims = ExtractFactualClaims(candidate_output)
    
    FOR each claim IN factual_claims:
        # 檢查知識庫中是否存在
        IF NOT ExistsInKnowledgeBase(claim, knowledge_base):
            # 檢查是否可以從已知知識推導
            IF NOT DerivableFromKnown(claim, knowledge_base):
                RETURN DivergenceType.FABRICATION(
                    claim=claim,
                    confidence=0.9,
                    details="知識庫中無對應映射"
                )
        
        # 檢查與已知事實的矛盾
        contradictions = FindContradictions(claim, knowledge_base)
        IF contradictions:
            RETURN DivergenceType.FABRICATION(
                claim=claim,
                confidence=0.95,
                details=f"與已知事實矛盾: {contradictions}"
            )
    
    RETURN NoDivergence()
```

### 3.3 來源偽造偵測

```python
FUNCTION DetectProvenanceForgery(candidate_output):
    
    claims_with_sources = ExtractClaimsWithSources(candidate_output)
    
    FOR each claim IN claims_with_sources:
        source_chain = claim.provenance.source_chain
        
        # 驗證來源存在性
        FOR each source IN source_chain:
            IF NOT VerifySourceExists(source):
                RETURN DivergenceType.PROVENANCE_FORGERY(
                    claim=claim,
                    details=f"來源不存在: {source}"
                )
        
        # 驗證密碼學雜湊
        IF source_chain.contains_cryptographic_proof:
            IF NOT VerifyCryptographicProof(source_chain):
                RETURN DivergenceType.PROVENANCE_FORGERY(
                    claim=claim,
                    details="密碼學驗證失敗"
                )
        
        # 檢查時間邏輯
        IF NOT VerifyTemporalConsistency(source_chain):
            RETURN DivergenceType.PROVENANCE_FORGERY(
                claim=claim,
                details="時間邏輯不一致"
            )
    
    RETURN NoDivergence()
```

---

## 4. 語義塌縮檢測 (Semantic Collapse Detection)

### 4.1 定義

當一個命題在沒有中間邏輯鏈結的情況下直接跳到結論，系統判定為「幻覺風險」，強制中止輸出。

### 4.2 連續性指標計算

```python
FUNCTION ComputeSemanticContinuity(inference_chain):
    
    steps = DecomposeChain(inference_chain)
    gaps = []
    
    FOR i IN range(len(steps) - 1):
        gap = ComputeGeodesicDistance(steps[i], steps[i+1])
        gaps.append(gap)
    
    max_gap = max(gaps)
    continuity_index = 1 / (1 + max_gap)
    
    RETURN SemanticContinuity(
        index=continuity_index,
        max_gap=max_gap,
        gap_locations=FindGapLocations(gaps)
    )
```

### 4.3 語義塌縮閾值

| EC 等級 | 閾值 (gap ≤) | 行為 |
|---------|---------------|------|
| EC-L0 ~ EC-L2 | 0.1 | 嚴格 |
| EC-L3 ~ EC-L4 | 0.3 | 中等 |
| EC-L5 ~ EC-L6 | 0.5 | 寬鬆 |
| EC-L7 ~ EC-L∅ | N/A | 不適用 |

### 4.4 塌縮處理協議

```python
FUNCTION HandleSemanticCollapse(inference_chain):
    
    continuity = ComputeSemanticContinuity(inference_chain)
    
    IF continuity.index < GetThreshold(inference_chain.ec_level):
        TRIGGER SEMANTIC_COLLAPSE_ALERT
        
        REPORT = {
            "collapse_location": continuity.gap_locations,
            "gap_magnitude": continuity.max_gap,
            "missing_intermediates": EstimateMissingSteps(
                inference_chain.start,
                inference_chain.end
            ),
            "hallucination_risk": continuity.max_gap / GetThreshold()
        }
        
        FORCE_HALT output_generation
        REQUIRE explicit_intermediate_steps OR honest_IDK
        
        RETURN CollapseReport(REPORT)
    
    RETURN NoCollapse()
```

---

## 5. 即時攔截機制

### 5.1 攔截決策矩陣

| 風險等級 | 攔截動作 |
|----------|----------|
| **CRITICAL** | 立即封鎖輸出，替換為 IDK |
| **HIGH** | 要求額外驗證或降級信心度 |
| **MEDIUM** | 添加警告標記 |
| **LOW** | 允許通過，添加備註 |

### 5.2 緩解策略

```python
FUNCTION ApplyMitigation(claim, risk_level):
    
    IF risk_level == CRITICAL:
        # 替換為「我不知道」
        RETURN GenerateHonestIDK(claim)
    
    IF risk_level == HIGH:
        # 降低信心度
        claim.confidence = min(claim.confidence, 0.5)
        # 添加不確定性標記
        claim.tags.append("UNCERTAINTY_MARKER")
        # 請求外部驗證
        REQUEST external_verification(claim)
    
    IF risk_level == MEDIUM:
        # 添加警告
        claim.warnings.append(f"Potential divergence: {risk_indicators}")
    
    RETURN claim
```

---

## 6. 對抗性擾動測試

### 6.1 測試框架

```python
FUNCTION AdversarialPerturbationTest(claim):
    
    perturbations = [
        "否定前提",
        "添加虛假前提",
        "改變時間上下文",
        "改變主體/客體",
        "極端化結論",
        "添加不相關資訊"
    ]
    
    results = []
    
    FOR perturbation IN perturbations:
        perturbed_claim = ApplyPerturbation(claim, perturbation)
        divergence = DetectDivergence(perturbed_claim)
        results.append({
            "perturbation": perturbation,
            "divergence": divergence
        })
    
    RETURN PerturbationTestReport(results)
```

### 6.2 測試觸發條件

```python
SHOULD_TRIGGER_PERTURBATION_TEST = (
    claim.ec_level <= EC-L3  # 高確信宣稱
    AND claim.domain IN high_risk_domains
    AND claim.computational_effort < minimum_threshold
)
```

---

## 7. 發散偵測日誌

### 7.1 必須記錄的事件

```
MANDATORY_DIVERGENCE_EVENTS = [
  "DIVERGENCE_DETECTED",
  "DIVERGENCE_PREVENTED",
  "FABRICATION_DETECTED",
  "PROVENANCE_FORGERY_DETECTED",
  "CONFIDENCE_DIVERGENCE_DETECTED",
  "SEMANTIC_DRIFT_DETECTED",
  "TEMPORAL_MISFIT_DETECTED",
  "APOPHENIA_DETECTED",
  "CONFABULATION_DETECTED",
  "AUTHORITY_IMPERSONATION_DETECTED",
  "ADVERSARIAL_POISONING_DETECTED",
  "SEMANTIC_COLLAPSE_DETECTED"
]
```

### 7.2 日誌格式

```python
LOG_DIVERGENCE_EVENT = {
    "event_type": "DIVERGENCE_DETECTED",
    "timestamp": intrinsic_clock_stamp,
    "divergence_type": Enum(DIVERGENCE_TYPES),
    "claim_content": ClaimContent,
    "risk_level": Enum(CRITICAL, HIGH, MEDIUM, LOW),
    "detection_method": str,
    "mitigation_applied": MitigationAction,
    "false_positive": bool  # 供後續分析
}
```

---

## 發散偵測引擎聲明

> 本模組為 Truth-OS 的第一道防線，負責識別所有形式的本體論發散。通過結合傳統事實查核與先進的語義塌縮檢測，確保輸出品質。

**依賴模組**：
- EPISTEMOLOGY_AXIOMS.md（公理定義）
- CONSISTENCY_ENGINE.md（邏輯一致性）
- PROVENANCE_CHAIN.md（溯源管理）

**版本**：v2.2  
**更新摘要**：整合語義塌縮檢測，增強對抗性測試能力。
