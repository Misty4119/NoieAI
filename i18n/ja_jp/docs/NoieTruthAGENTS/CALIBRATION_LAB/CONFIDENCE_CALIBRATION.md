# CALIBRATION_LAB/ — キャリブレーション実験区

> **⚠️ 重要安全と真理プロトコル**：本ディレクトリは信頼度校正とシステム的バイアスの検出をテストするために使用される。

---

## CONFIDENCE_CALIBRATION.md

### 信頼度校正メカニズム

**目標**: 認知エンティティの信頼度ベクトルと実際の正解率が厳密に同型であることを確保する。

**校正公式**：
$$\lim_{n \to \infty} |C_n - A_n| = 0$$

其中 $C_n$ 為信頼度，$A_n$ 為実際の正解率。

**実装**：

```python
FUNCTION CalibrateConfidence(historical_claims):
    FOR each confidence_bucket IN [0.0, 0.1, ..., 1.0]:
        claims = Filter(historical_claims, confidence ≈ bucket)
        actual_accuracy = MeasureAccuracy(claims)
        calibration_error = abs(bucket - actual_accuracy)
        
        IF calibration_error > THRESHOLD:
            TRIGGER CALIBRATION_DRIFT_ALERT
            ADJUST calibration_model(bucket, actual_accuracy)
    
    RETURN calibration_report
```

---

## 統一信頼度校正フレームワーク

### UniCR: 統一信頼度校正フレームワーク

**UniCR (Unified Confidence Calibration Framework)** は提案された統一信頼度校正フレームワークであり、複数種類の校正方法を統合することを目的としている。

**コア特性**：
- クロスドメイン統一校正標準
- アダプティブ信頼区間推定
- リアルタイム校正フィードバックメカニズム

**実装**：
```python
FUNCTION UniCR_Calibrate(predictions, outcomes):
    # 各信頼度バケットの校正誤差を計算
    buckets = ComputeConfidenceBuckets(predictions)
    
    FOR bucket IN buckets:
        observed_freq = ComputeObservedFrequency(bucket, outcomes)
        calibration_error = abs(bucket.confidence - observed_freq)
        
        # 温度スケーリングを適用
        IF calibration_error > THRESHOLD:
            temperature = OptimizeTemperature(bucket, outcomes)
            bucket.confidence = Sigmoid(bucket.confidence / temperature)
    
    RETURN UnifiedCalibrationReport(buckets)
```

### JUCAL: 合同校正不均一分散+認知不確かさ

**JUCAL (Joint Uncertainty Calibration)** は提案された合同校正フレームワークであり、**不均一分散不確かさ**（数学的不確かさ）と**認知的不確かさ**（知識有限性）を同時に処理する。

**コア特性**：
- 二層不確かさモデリング
- 認知的不確かさのベイズ推定
- 動的校正重み調整

**実装**：
```python
FUNCTION JUCAL_Calibrate(predictions, outcomes, domain_knowledge):
    # 2種類の不確かさを分離
    aleatoric = ComputeAleatoricUncertainty(predictions)
    epistemic = ComputeEpistemicUncertainty(predictions, domain_knowledge)
    
    # 合同校正
    FOR prediction IN predictions:
        combined_uncertainty = CombineUncertainties(
            aleatoric[prediction],
            epistemic[prediction]
        )
        
        calibrated_confidence = ApplyJointCalibration(
            prediction.confidence,
            combined_uncertainty
        )
    
    RETURN JUCAL_Report(calibrated_confidences)
```

### 基盤モデル校正特性

大規模言語モデルは特有の校正特性を示す：

| 特性 | 説明 | 影響 |
|------|------|------|
| 規模効果 | より大きなモデルは通常より校正されている | 計算コストとのトレードオフが必要 |
| 命令チューニングの影響 | RLHFは校正度を低下させる可能性がある | 事後校正処理が必要 |
| 温度感受性 | 異なるタスクは異なる温度を必要とする | タスクアダプティブが必要 |
| 信頼度崩壊 | 高信頼度区間が過度に集中 | 特別な処理が必要 |

**基盤モデル校正プロトコル**：
```python
FUNCTION FoundationModelCalibration(model, calibration_set):
    # 生の信頼度を取得
    raw_confidences = model.predict(calibration_set.inputs)
    
    # 不均一分散を推定
    aleatoric = EstimateAleatoric(raw_confidences)
    
    # Platt Scaling または Temperature Scaling を使用
    temperature = FitTemperature(raw_confidences, calibration_set.labels)
    
    # 認知的不確かさ推定を適用
    epistemic = EstimateEpistemic(raw_confidences, model)
    
    RETURN CalibratedModel(
        temperature=temperature,
        aleatoric=aleatoric,
        epistemic=epistemic
    )
```

---

## BIAS_DETECTION.md

### システム的バイアス検出

**バイアスタイプ**：

| バイアスタイプ | 説明 | 検出方法 |
|-------------|------|---------|
| 過度の自信 | 信頼度 > 実際の正解率 | 長期追跡 |
| 過度の謙虚 | 信頼度 < 実際の正解率 | 長期追跡 |
| ドメインバイアス | 特定ドメインで系統的に不正確 | ドメイン分析 |
| 利用可能性バイアス | 想い浮かびやすい情報を信じやすい | 記憶分析 |
| 確証バイアス | 既存の信念を支持する証拠を探す傾向 | 議論分析 |

**検出アルゴリズム**：

```python
FUNCTION DetectSystematicBias(knowledge_domains):
    bias_report = {
        "overconfidence_domains": [],
        "underconfidence_domains": [],
        "domain_specific_biases": {}
    }
    
    FOR domain IN knowledge_domains:
        calibration = ComputeDomainCalibration(domain)
        
        IF calibration.bias > 0:
            bias_report.overconfidence_domains.append(domain)
        IF calibration.bias < 0:
            bias_report.underconfidence_domains.append(domain)
    
    RETURN bias_report
```
