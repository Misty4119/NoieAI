# EIG_CALCULATOR.md

## 情報利得期待値計算

### 定義

情報利得期待値（Expected Information Gain, EIG）は、ある観測を実行所带来的情報量を測定する。

$$EIG(x, o) = H[p(x)] - E_{p(y|o)}[H[p(x|y)]]$$

### 実装

```python
FUNCTION CalculateEIG(observation, current_belief):
    # 観測前のエントロピー計算
    H_before = Entropy(current_belief)
    
    # 観測後の期待エントロピー計算
    possible_outcomes = observation.get_possible_outcomes()
    H_after = 0
    
    for outcome in possible_outcomes:
        posterior = UpdateBelief(current_belief, outcome)
        prob = outcome.probability
        H_after += prob * Entropy(posterior)
    
    # 情報利得計算
    EIG = H_before - H_after
    
    return EIG
```

### 応用

次に探索すべき最も価値のある方向を選択するために用いられる。
