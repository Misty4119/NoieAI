# NONCOMMUTATIVE_DETECTION.md

## 非交換性偵測

### 定義

當觀測算符 $\hat{A}$ 與 $\hat{B}$ 滿足 $[\hat{A}, \hat{B}] \neq 0$ 時，觀測順序會改變結果：

$$\text{Measure}_A(\text{Measure}_B(\text{State})) \neq \text{Measure}_B(\text{Measure}_A(\text{State}))$$

### 偵測演算法

```python
FUNCTION DetectNonCommutativity(observation_pair):
    
    A, B = observation_pair
    
    # 計算對易子
    commutator = ComputeCommutator(A, B)
    
    IF commutator != 0:
        # 計算不確定性關係
        uncertainty = ComputeHeisenbergUncertainty(A, B)
        
        RETURN NonCommutativePair(
            operator_A=A,
            operator_B=B,
            commutator_value=commutator,
            uncertainty_relation=uncertainty,
            requires_order_declaration=True
        )
    
    RETURN CommutativePair(A, B)
```

### 不確定性關係

**Robertson-Schrödinger 擴展**：

$$\sigma^2_A \cdot \sigma^2_B \geq \frac{1}{4}|\langle[A, B]\rangle|^2 + \left(\frac{\langle \{A, B\} \rangle}{2} - \langle A \rangle \langle B \rangle\right)^2$$

其中：
- $\sigma_A, \sigma_B$ 為標準差
- $[A, B] = AB - BA$ 為對易子
- $\{A, B\} = AB + BA$ 為反對易子

### 處理協議

```python
FUNCTION HandleNonCommutativePair(pair):
    
    IF pair.requires_order_declaration:
        TRIGGER NONCOMMUTATIVE_ALERT
        MARK dependent_claims AS ORDER_SENSITIVE
        REQUIRE explicit_observation_order_declaration
        
        # 降低相關結論信心度
        FOR claim IN pair.dependent_claims:
            claim.confidence *= 0.8
            claim.warnings.append("觀測順序依賴")
```
