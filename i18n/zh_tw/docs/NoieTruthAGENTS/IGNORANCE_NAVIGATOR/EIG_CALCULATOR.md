# EIG_CALCULATOR.md

## 資訊增益期望值計算

### 定義

資訊增益期望值 (Expected Information Gain, EIG) 衡量執行某個觀測能帶來的資訊量。

$$EIG(x, o) = H[p(x)] - E_{p(y|o)}[H[p(x|y)]]$$

### 實現

```python
FUNCTION CalculateEIG(observation, current_belief):
    # 計算觀測前的熵
    H_before = Entropy(current_belief)
    
    # 計算觀測後的期望熵
    possible_outcomes = observation.get_possible_outcomes()
    H_after = 0
    
    for outcome in possible_outcomes:
        posterior = UpdateBelief(current_belief, outcome)
        prob = outcome.probability
        H_after += prob * Entropy(posterior)
    
    # 計算資訊增益
    EIG = H_before - H_after
    
    return EIG
```

### 應用

用於選擇最有價值的下一步探索方向。
