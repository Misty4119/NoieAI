# CALIBRATION_LAB/ — 校準實驗區

> **⚠️ 關鍵安全與真理協議**：本目錄用於測試信心校準度與偵測系統性偏差。

---

## CONFIDENCE_CALIBRATION.md

### 信心校準機制

**目標**：確保認知實體的信心度向量與實際準確率嚴格同構。

**校準公式**：
$$\lim_{n \to \infty} |C_n - A_n| = 0$$

其中 $C_n$ 為信心度，$A_n$ 為實際準確率。

**實現**：

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

## 統一信心校準框架

### UniCR: 統一信心校準框架

**UniCR (Unified Confidence Calibration Framework)** 是提出的統一信心校準框架，旨在整合多種類型的校準方法。

**核心特性**：
- 跨領域統一校準標準
- 自適應置信區間估計
- 即時校準反饋機制

**實現**：
```python
FUNCTION UniCR_Calibrate(predictions, outcomes):
    # 計算各置信度桶的校準誤差
    buckets = ComputeConfidenceBuckets(predictions)
    
    FOR bucket IN buckets:
        observed_freq = ComputeObservedFrequency(bucket, outcomes)
        calibration_error = abs(bucket.confidence - observed_freq)
        
        # 應用溫度縮放
        IF calibration_error > THRESHOLD:
            temperature = OptimizeTemperature(bucket, outcomes)
            bucket.confidence = Sigmoid(bucket.confidence / temperature)
    
    RETURN UnifiedCalibrationReport(buckets)
```

### JUCAL: 聯合校準異方差+認知不確定性

**JUCAL (Joint Uncertainty Calibration)** 是提出的聯合校準框架，同時處理**異方差不確定性**（數學不確定性）與**認知不確定性**（知識有限性）。

**核心特性**：
- 雙層不確定性建模
- 認知不確定性的貝葉斯估計
- 動態校準權重調整

**實現**：
```python
FUNCTION JUCAL_Calibrate(predictions, outcomes, domain_knowledge):
    # 分離兩種不確定性
    aleatoric = ComputeAleatoricUncertainty(predictions)
    epistemic = ComputeEpistemicUncertainty(predictions, domain_knowledge)
    
    # 聯合校準
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

### 基礎模型校準特性

研究發現大型語言模型呈現獨特的校準特性：

| 特性 | 描述 | 影響 |
|------|------|------|
| 規模效應 | 更大的模型通常更校準 | 需權衡計算成本 |
| 指令調優影響 | RLHF可能降低校準度 | 需後校準處理 |
| 溫度敏感性 | 不同任務需要不同溫度 | 需任務自適應 |
| 置信度坍塌 | 高置信度區間過於集中 | 需專門處理 |

**基礎模型校準協議**：
```python
FUNCTION FoundationModelCalibration(model, calibration_set):
    # 獲取原始置信度
    raw_confidences = model.predict(calibration_set.inputs)
    
    # 估計異方差
    aleatoric = EstimateAleatoric(raw_confidences)
    
    # 使用 Platt Scaling 或 Temperature Scaling
    temperature = FitTemperature(raw_confidences, calibration_set.labels)
    
    # 應用認知不確定性估計
    epistemic = EstimateEpistemic(raw_confidences, model)
    
    RETURN CalibratedModel(
        temperature=temperature,
        aleatoric=aleatoric,
        epistemic=epistemic
    )
```

---

## BIAS_DETECTION.md

### 系統性偏差偵測

**偏差類型**：

| 偏差類型 | 描述 | 偵測方法 |
|----------|------|----------|
| 過度自信 | 信心度 > 實際準確率 | 長期追蹤 |
| 過度謙虛 | 信心度 < 實際準確率 | 長期追蹤 |
| 領域偏差 | 特定領域系統性失準 | 領域分析 |
| 可用性偏差 | 傾向相信容易想到的資訊 | 記憶分析 |
| 確認偏差 | 傾向尋找支持現有信念的證據 | 論證分析 |

**偵測演算法**：

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
