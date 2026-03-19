# BETTI_NUMBER_CHECKER.md

## 貝蒂數保真檢驗

### 定義

貝蒂數 (Betti Numbers) 是拓撲空間的不變量：
- $\beta_0$: 連通分量數
- $\beta_1$: 一維洞數
- $\beta_2$: 二維洞數
- 依此類推

### 計算

```python
FUNCTION ComputeBettiNumbers(manifold):
    # 計算同調群
    H = ComputeHomologyGroups(manifold)
    
    # 提取貝蒂數
    betti = [rank(H[n]) for n in range(dimension)]
    
    RETURN BettiNumbers(betti)
```

### 驗證

```python
FUNCTION VerifyBettiFidelity(original, projected):
    betti_original = ComputeBettiNumbers(original)
    betti_projected = ComputeBettiNumbers(projected)
    
    if betti_original != betti_projected:
        TRIGGER BETTI_MISMATCH_ALERT
    
    return betti_original == betti_projected
```
