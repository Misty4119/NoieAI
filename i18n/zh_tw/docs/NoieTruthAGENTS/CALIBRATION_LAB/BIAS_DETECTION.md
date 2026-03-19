# BIAS_DETECTION.md

## 系統性偏差偵測

### 偏差類型定義

| 偏差類型 | 符號描述 | 典型表現 |
|----------|----------|----------|
| **過度自信** | C > A 系統性 | 高估自己的能力 |
| **過度謙虛** | C < A 系統性 | 低估自己的能力 |
| **領域偏差** | 特定領域 C ≠ A | 某些領域系統性失準 |
| **可用性偏差** | 傾向記住易獲取資訊 | 忽略罕見但重要的資訊 |
| **確認偏差** | 傾向支持現有信念 | 忽視反例 |
| **錨定偏差** | 過度依賴第一資訊 | 後續判斷受初始值影響 |
| **後見之明偏差** | 認為過去事件可預測 | 低估隨機性 |

### 偵測演算法

```python
FUNCTION DetectBiases(claim_history):
    
    biases_detected = {}
    
    # 1. 過度自信偵測
    overconfidence = DetectOverconfidence(claim_history)
    biases_detected["overconfidence"] = overconfidence
    
    # 2. 過度謙虛偵測
    underconfidence = DetectUnderconfidence(claim_history)
    biases_detected["underconfidence"] = underconfidence
    
    # 3. 領域偏差偵測
    domain_biases = DetectDomainBiases(claim_history)
    biases_detected["domain_biases"] = domain_biases
    
    # 4. 可用性偏差偵測
    availability_bias = DetectAvailabilityBias(claim_history)
    biases_detected["availability"] = availability_bias
    
    # 5. 確認偏差偵測
    confirmation_bias = DetectConfirmationBias(claim_history)
    biases_detected["confirmation"] = confirmation_bias
    
    RETURN BiasReport(biases_detected)
```

### 偏差校正

```python
FUNCTION CorrectBiases(bias_report):
    
    corrections = {}
    
    IF bias_report.overconfidence.detected:
        corrections["confidence_adjustment"] = -bias_report.overconfidence.magnitude
        NOTIFY "系統性過度自信偵測，信心度已調整"
    
    IF bias_report.underconfidence.detected:
        corrections["confidence_adjustment"] = +bias_report.underconfidence.magnitude
        NOTIFY "系統性過度謙虛偵測，信心度已調整"
    
    FOR domain IN bias_report.domain_biases:
        corrections[domain] = CalculateDomainCorrection(domain)
    
    RETURN corrections
```
