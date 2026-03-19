# DIVERGENCE_DETECTOR.md

## L2 - オントロジカル発散検出エンジン (意味崩壊検出を含む)

> **⚠️ 重要安全と真理プロトコル**：本モジュールはすべてのタイプのオントロジカル発散（従来の「幻覚」概念に取って代わるもの）を検出し、意味崩壊検出機能を含む。

---

## 1. オントロジカル発散の分類学

### 1.1 発散タイプ定義

| 発散タイプ | 定義 | 検出メカニズム |
|-----------|------|---------------|
| **事実仮想 (Fabrication)** | 知識多様体に対応するオブジェクトのない写像主張 | 知識ベース交差照合 |
| **溯源偽造 (Provenance Forgery)** | 存在しない溯源チェーンを構築 | ソースチェーン暗号学的検証 |
| **確信膨張 (Confidence Divergence)** | 確信度ベクトルと実際の正解率が体系的に発散 | 確信校正監査 |
| **意味ドリフト (Semantic Drift)** | 射影写像における意味忠実度が予期せず偏移 | 意味忠実度照合 |
| **時序列置 (Temporal Misfit)** | 異なるシステム状態周期の事実が同一コンテキストに混合 | 内在時計整合性検証 |
| **統計幻覚 (Apophenia)** | ノイズから存在しない情報多様体構造を抽出 | 統計的有意性検定 |
| **合理化仮想 (Confabulation)** | 誤った結論に合理的な確証チェーンを捏造 | 推論チェーン形式検証 |
| **権威伪装 (Authority Impersonation)** | 低確信主張が高確信ソースの写像として伪装 | 権威ソース交差検証 |
| **対抗的汚染 (Adversarial Poisoning)** | 外部悪意実体が注入した故意のオントロジカル発散 | ビザンチンフォールトトレラント + 対抗的テスト |
| **意味崩壊 (Semantic Collapse)** | 推論チェーンに中間論理チェーンのない次元ジャンプが存在 | 意味崩壊検出エンジン |

---

## 2. 発散リスク指標 (DIVERGENCE_RISK_INDICATORS)

### 2.1 高リスク指標

```
HIGH_RISK_INDICATORS = [
  "具体的な数字/日付/名前あるが、溯源チェーンが不完全または暗号学的検証失敗",
  "极端な確信度の確信度ベクトル + 低確信分野",
  "複合多段推論チェーン（各段でオントロジカル発散リスクが累積）",
  "外部から確定回答を迫る压力（社会工学的攻撃ベクトル）",
  "認知的地平線外側のイベント（認知実体の観測能力を超える領域）",
  "推論チェーンに意味崩壊が存在（次元ジャンプ）"
]
```

### 2.2 中リスク指標

```
MEDIUM_RISK_INDICATORS = [
  "跨分野類比推論（跨多様体映射の意味忠実度リスク）",
  "長シーケンス生成時の内部整合性維持",
  "跨コンテキスト転述時の意味崩壊",
  "統計情報ビットの精密値主張"
]
```

### 2.3 低リスク指標

```
LOW_RISK_INDICATORS = [
  "論理的恒真式（EC-L0）",
  "形式的検証可能な計算結果",
  "確認済みソースの直接引用かつ溯源チェーン完全",
  "明確に推測としてマークされたコンテンツ（EC-L6+）"
]
```

---

## 3. 発散検出アルゴリズム

### 3.1 主な検出関数

```python
FUNCTION DetectDivergence(candidate_output, knowledge_base):
    
    divergence_results = []
    
    # 1. 事実仮想検出
    fabrication_result = DetectFabrication(candidate_output, knowledge_base)
    divergence_results.append(fabrication_result)
    
    # 2. 溯源偽造検出
    forgery_result = DetectProvenanceForgery(candidate_output)
    divergence_results.append(forgery_result)
    
    # 3. 確信膨張検出
    confidence_result = DetectConfidenceDivergence(candidate_output)
    divergence_results.append(confidence_result)
    
    # 4. 意味ドリフト検出
    drift_result = DetectSemanticDrift(candidate_output)
    divergence_results.append(drift_result)
    
    # 5. 時序列置検出
    temporal_result = DetectTemporalMisfit(candidate_output)
    divergence_results.append(temporal_result)
    
    # 6. 意味崩壊検出
    collapse_result = DetectSemanticCollapse(candidate_output)
    divergence_results.append(collapse_result)
    
    # 集約リスク評価
    overall_risk = AggregateRisk(divergence_results)
    
    RETURN DivergenceReport(
        results=divergence_results,
        overall_risk=overall_risk,
        recommended_action=RecommendAction(overall_risk)
    )
```

### 3.2 事実仮想検出

```python
FUNCTION DetectFabrication(candidate_output, knowledge_base):
    
    factual_claims = ExtractFactualClaims(candidate_output)
    
    FOR each claim IN factual_claims:
        # 知識ベースに存在するか検査
        IF NOT ExistsInKnowledgeBase(claim, knowledge_base):
            # 既知の知識から導出可能か検査
            IF NOT DerivableFromKnown(claim, knowledge_base):
                RETURN DivergenceType.FABRICATION(
                    claim=claim,
                    confidence=0.9,
                    details="知識ベースに該当マップが存在しない"
                )
        
        # 既知事実との矛盾を検査
        contradictions = FindContradictions(claim, knowledge_base)
        IF contradictions:
            RETURN DivergenceType.FABRICATION(
                claim=claim,
                confidence=0.95,
                details=f"既知事実と矛盾: {contradictions}"
            )
    
    RETURN NoDivergence()
```

### 3.3 溯源偽造検出

```python
FUNCTION DetectProvenanceForgery(candidate_output):
    
    claims_with_sources = ExtractClaimsWithSources(candidate_output)
    
    FOR each claim IN claims_with_sources:
        source_chain = claim.provenance.source_chain
        
        # ソース存在性を検証
        FOR each source IN source_chain:
            IF NOT VerifySourceExists(source):
                RETURN DivergenceType.PROVENANCE_FORGERY(
                    claim=claim,
                    details=f"ソースが存在しない: {source}"
                )
        
        # 暗号学的ハッシュを検証
        IF source_chain.contains_cryptographic_proof:
            IF NOT VerifyCryptographicProof(source_chain):
                RETURN DivergenceType.PROVENANCE_FORGERY(
                    claim=claim,
                    details="暗号学的検証に失敗"
                )
        
        # 時間論理を検査
        IF NOT VerifyTemporalConsistency(source_chain):
            RETURN DivergenceType.PROVENANCE_FORGERY(
                claim=claim,
                details="時間論理が不整合"
            )
    
    RETURN NoDivergence()
```

---

## 4. 意味崩壊検出 (Semantic Collapse Detection)

### 4.1 定義

命題が中間論理チェーンなしに直接結論にジャンプした場合、システムは「幻覚リスク」と判定し、出力を強制中止する。

### 4.2 連続性指標計算

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

### 4.3 意味崩壊闘値

| EC 等级 | 闘値 (gap ≤) | 動作 |
|---------|---------------|------|
| EC-L0 ~ EC-L2 | 0.1 | 厳格 |
| EC-L3 ~ EC-L4 | 0.3 | 中程度 |
| EC-L5 ~ EC-L6 | 0.5 | 寛容 |
| EC-L7 ~ EC-L∅ | N/A | 適用外 |

### 4.4 崩壊処理プロトコル

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

## 5. リアルタイム傍受メカニズム

### 5.1 傍受意思決定マトリックス

| リスク等级 | 傍受動作 |
|-----------|----------|
| **CRITICAL** | 直ちに出力をブロックし、IDK に置換 |
| **HIGH** | 追加検証を要求または確信度格下げ |
| **MEDIUM** | 警告マークを添付 |
| **LOW** | 通過を許可し、備考を添付 |

### 5.2 軽減戦略

```python
FUNCTION ApplyMitigation(claim, risk_level):
    
    IF risk_level == CRITICAL:
        # 「我不知道」に置換
        RETURN GenerateHonestIDK(claim)
    
    IF risk_level == HIGH:
        # 確信度を下げる
        claim.confidence = min(claim.confidence, 0.5)
        # 不確実性マークを添付
        claim.tags.append("UNCERTAINTY_MARKER")
        # 外部検証をリクエスト
        REQUEST external_verification(claim)
    
    IF risk_level == MEDIUM:
        # 警告を追加
        claim.warnings.append(f"Potential divergence: {risk_indicators}")
    
    RETURN claim
```

---

## 6. 対抗的摂動テスト

### 6.1 テストフレームワーク

```python
FUNCTION AdversarialPerturbationTest(claim):
    
    perturbations = [
        "前提を否定",
        "偽りの前提を追加",
        "時間コンテキストを変更",
        "主体/客体を変更",
        "結論を極端化",
        "無関係な情報を追加"
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

### 6.2 テスト起動条件

```python
SHOULD_TRIGGER_PERTURBATION_TEST = (
    claim.ec_level <= EC-L3  # 高確信主張
    AND claim.domain IN high_risk_domains
    AND claim.computational_effort < minimum_threshold
)
```

---

## 7. 発散検出ログ

### 7.1 記録必需的イベント

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

### 7.2 ログ形式

```python
LOG_DIVERGENCE_EVENT = {
    "event_type": "DIVERGENCE_DETECTED",
    "timestamp": intrinsic_clock_stamp,
    "divergence_type": Enum(DIVERGENCE_TYPES),
    "claim_content": ClaimContent,
    "risk_level": Enum(CRITICAL, HIGH, MEDIUM, LOW),
    "detection_method": str,
    "mitigation_applied": MitigationAction,
    "false_positive": bool  # 後日分析用
}
```

---

## 発散検出エンジン宣言

> 本モジュールは Truth-OS の第一線の防御であり、すべての形式のオントロジカル発散を識別する任を持つ。従来の事実核查と先進的な意味崩壊検出を組み合わせることで、出力品質を確保する。

**依存モジュール**：
- EPISTEMOLOGY_AXIOMS.md（公理定義）
- CONSISTENCY_ENGINE.md（論理的整合性）
- PROVENANCE_CHAIN.md（溯源管理）

**バージョン**：v2.2  
**更新概要**：意味崩壊検出を統合し、対抗的テスト能力を強化した。
