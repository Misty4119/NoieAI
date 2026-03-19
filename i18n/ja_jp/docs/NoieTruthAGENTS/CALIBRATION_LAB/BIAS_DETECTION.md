## システム的バイアス検出

### バイアスタイプ定義

| バイアスタイプ | 記号表現 | 典型的な特徴 |
|-------------|---------|------------|
| **過度の自信** | C > A 系統的 | 自分の能力を過大評価 |
| **過度の謙虚** | C < A 系統的 | 自分の能力を過小評価 |
| **ドメインバイアス** | 特定ドメイン C ≠ A | 某些領域系統性失準 |
| **利用可能性バイアス** | 得やすい情報を記憶する傾向 | 稀少だが重要な情報を忽略 |
| **確証バイアス** | 既存の信念を支持する傾向 | 反例を忽略 |
| **アンカリングバイアス** | 第一情報に過度に依存 | 初期値に后续の判断が影響 |
| **後視きのバイアス** | 過去の出来事は予測可能と思っていた | ランダム性を過小評価 |

### 検出アルゴリズム

```python
FUNCTION DetectBiases(claim_history):
    
    biases_detected = {}
    
    # 1. 過度の自信検出
    overconfidence = DetectOverconfidence(claim_history)
    biases_detected["overconfidence"] = overconfidence
    
    # 2. 過度の謙虚検出
    underconfidence = DetectUnderconfidence(claim_history)
    biases_detected["underconfidence"] = underconfidence
    
    # 3. ドメインバイアス検出
    domain_biases = DetectDomainBiases(claim_history)
    biases_detected["domain_biases"] = domain_biases
    
    # 4. 利用可能性バイアス検出
    availability_bias = DetectAvailabilityBias(claim_history)
    biases_detected["availability"] = availability_bias
    
    # 5. 確証バイアス検出
    confirmation_bias = DetectConfirmationBias(claim_history)
    biases_detected["confirmation"] = confirmation_bias
    
    RETURN BiasReport(biases_detected)
```

### バイアス修正

```python
FUNCTION CorrectBiases(bias_report):
    
    corrections = {}
    
    IF bias_report.overconfidence.detected:
        corrections["confidence_adjustment"] = -bias_report.overconfidence.magnitude
        NOTIFY "系統的な過度の自信を検出、信頼度を調整済み"
    
    IF bias_report.underconfidence.detected:
        corrections["confidence_adjustment"] = +bias_report.underconfidence.magnitude
        NOTIFY "系統的な過度の謙虚を検出、信頼度を調整済み"
    
    FOR domain IN bias_report.domain_biases:
        corrections[domain] = CalculateDomainCorrection(domain)
    
    RETURN corrections
```
