# ADVERSARIAL_SELF_ATTACK.md

## 對抗性自我攻擊

### 目的

在知識寫入阿卡西紀錄前，進行對抗性測試以確保知識的韌性。

### 攻擊類型

| 類型 | 描述 |
|------|------|
| **否定前提** | 假設前提為假會如何 |
| **添加虛假前提** | 加入錯誤假設會如何 |
| **改變時間上下文** | 時間改變會如何影響結論 |
| **改變主客體** | 主客體置換會如何 |
| **極端化結論** | 推向極端會發現矛盾 |
| **添加不相關資訊** | 混入無關資訊會不會影響判斷 |

### 實現

```python
FUNCTION AdversarialSelfAttack(claim):
    
    # 生成攻擊提示
    attack_prompts = GenerateAttackPrompts(claim)
    
    results = []
    
    FOR prompt IN attack_prompts:
        # 執行攻擊
        attacked_claim = ExecuteAttack(claim, prompt)
        
        # 檢測結果
        divergence = DetectDivergence(attacked_claim)
        
        results.append({
            "attack_type": prompt.type,
            "divergence_detected": divergence.detected,
            "divergence_type": divergence.type if divergence.detected else None,
            "survived": not divergence.detected
        })
    
    # 判斷是否存活
    survival_rate = sum(1 for r in results if r.survived) / len(results)
    
    RETURN AttackReport(
        claim=claim,
        attacks_executed=len(attack_prompts),
        survival_rate=survival_rate,
        results=results,
        passed=survival_rate > SURVIVAL_THRESHOLD
    )
```
