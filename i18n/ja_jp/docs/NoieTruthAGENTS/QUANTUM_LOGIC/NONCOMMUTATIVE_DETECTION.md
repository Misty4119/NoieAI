# NONCOMMUTATIVE_DETECTION.md

## 非可換性検出

### 定義

観測演算子 $\hat{A}$ と $\hat{B}$ が $[\hat{A}, \hat{B}] \neq 0$ を満たす場合、観測順序が結果を変更する：

$$\text{Measure}_A(\text{Measure}_B(\text{State})) \neq \text{Measure}_B(\text{Measure}_A(\text{State}))$$

### 検出アルゴリズム

```python
FUNCTION DetectNonCommutativity(observation_pair):
    
    A, B = observation_pair
    
    # 交換子の計算
    commutator = ComputeCommutator(A, B)
    
    IF commutator != 0:
        # 不確定性関係の計算
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

### 不確定性関係

**Robertson-Schrödinger 拡張**：

$$\sigma^2_A \cdot \sigma^2_B \geq \frac{1}{4}|\langle[A, B]\rangle|^2 + \left(\frac{\langle \{A, B\} \rangle}{2} - \langle A \rangle \langle B \rangle\right)^2$$

其中：
- $\sigma_A, \sigma_B$ は標準偏差
- $[A, B] = AB - BA$ は交換子
- $\{A, B\} = AB + BA$ は反交換子

### 処理プロトコル

```python
FUNCTION HandleNonCommutativePair(pair):
    
    IF pair.requires_order_declaration:
        TRIGGER NONCOMMUTATIVE_ALERT
        MARK dependent_claims AS ORDER_SENSITIVE
        REQUIRE explicit_observation_order_declaration
        
        # 関連結論の信頼度を下げる
        FOR claim IN pair.dependent_claims:
            claim.confidence *= 0.8
            claim.warnings.append("観測順序依存")
```
