# NEWS_DECAY.md

## ニュース減衰領域知識減衰律

### 領域特性

| 特性 | 説明 |
|------|------|
| **情報代謝率** | 极高 |
| **即時性** | 時間の敏感性 が極めて強い |
| **事実安定性** | 事実が急速に時代錯誤になる可能性 |
| **ソース多样性** | 信頼性の差が大きい |

### 減衰定数

$$\lambda^*_{\text{news}} \approx 0.8 - 1.0$$

### ニュースサブタイプ減衰特徴

| サブタイプ | λ* 範囲 | 半減期 |
|--------|---------|--------|
| 即時ニュース | 0.9 - 1.0 | 数時間 |
| 当日ニュース | 0.7 - 0.9 | 1-3 日 |
| 专题報道 | 0.5 - 0.7 | 1-2 週間 |
| 評論分析 | 0.3 - 0.5 | 1-3 ヶ月 |
| 深掘り調査 | 0.2 - 0.4 | 数ヶ月-数年 |

### 減衰トリガー条件

```python
NEWS_DECAY_TRIGGERS = [
    "新規イベント展開",
    "事実澄清/更正",
    "関連ニュース反転",
    "公式声明发布",
    "時間経過が閾値を超える"
]
```

### 計算公式

```python
FUNCTION ComputeNewsValidity(claim, current_intrinsic_clock):
    clock_delta = current_intrinsic_clock - claim.ν_stamp
    decay = exp(-lambda_news * clock_delta)
    source_reliability = GetSourceReliability(claim.source)
    RETURN min(decay * source_reliability * claim.corroboration_factor, 1.0)
```
