# LEGAL_DECAY.md

## 法律領域知識衰減律

### 領域特性

| 特性 | 描述 |
|------|------|
| **法規穩定性** | 相對穩定但有修法週期 |
| **判例累積效應** | 舊判例可能被新判例推翻 |
| **時效性** | 法律有追訴時效 |
| **地域性** | 不同法域有不同規定 |

### 衰減常數

$$\lambda^*_{\text{legal}} \approx 0.05 - 0.15$$

### 法律子領域衰減特徵

| 子領域 | λ* 範圍 | 衰減觸發 |
|--------|---------|----------|
| 憲法 | 0.01 - 0.05 | 憲法修正 |
| 刑法 | 0.02 - 0.08 | 刑罰修正 |
| 民法 | 0.03 - 0.10 | 民法修正 |
| 行政法 | 0.05 - 0.15 | 法規修正 |
| 國際法 | 0.02 - 0.08 | 条約修訂 |

### 衰減觸發條件

```python
LEGAL_DECAY_TRIGGERS = [
    "法律修正通過",
    "司法解釋變更",
    "判例逆轉",
    "時效完成",
    "法域變更",
    "條約退出/加入"
]
```

### 計算公式

```python
FUNCTION ComputeLegalValidity(claim, current_date, jurisdiction):
    time_delta = ComputeTimeSinceClaim(current_date, claim.effective_date)
    law_changes = CountLawChanges(claim.law_reference, jurisdiction)
    decay = exp(-lambda_legal * (time_delta + law_changes))
    RETURN min(decay * claim.original_reliability, 1.0)
```
