# LEGAL_DECAY.md

## 法的減衰領域知識減衰律

### 領域特性

| 特性 | 説明 |
|------|------|
| **法規安定性** | 比較的安定だが、法改正サイクルがある |
| **判例蓄積効果** | 古い判例は新判例によって覆される可能性 |
| **時効性** | 法律には訴追時効がある |
| **領域性** | 異なる法域有不同的規定 |

### 減衰定数

$$\lambda^*_{\text{legal}} \approx 0.05 - 0.15$$

### 法律サブ領域減衰特徴

| サブ領域 | λ* 範囲 | 減衰トリガー |
|--------|---------|----------|
| 憲法 | 0.01 - 0.05 | 憲法改正 |
| 刑法 | 0.02 - 0.08 | 刑罰改正 |
| 民法 | 0.03 - 0.10 | 民法改正 |
| 行政法 | 0.05 - 0.15 | 法規改正 |
| 国際法 | 0.02 - 0.08 | 条約改正 |

### 減衰トリガー条件

```python
LEGAL_DECAY_TRIGGERS = [
    "法律改正成立",
    "司法解釈変更",
    "判例逆転",
    "時効完成",
    "法域変更",
    "条約脱退/加盟"
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
