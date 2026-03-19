# ADVERSARIAL_SELF_ATTACK.md

## 対抗的自己攻撃

### 目的

知識をアカシックレコードに書き込む前に、知識の堅牢性を確保するための対抗的テストを実施する。

### 攻撃タイプ

| タイプ | 説明 |
|------|------|
| **否定前提** | 前提が偽であると仮定するとどうなるか |
| **偽前提の追加** | 誤った仮定を加えるとどうなるか |
| **時間コンテキストの変更** | 時間が変わると結論にどう影響するか |
| **主語・目的語の変更** | 主語・目的語を入れ替えるとどうなるか |
| **結論の極端化** | 極端にすると矛盾が発見できる |
| **無関係な情報の追加** | 無関係な情報を混入すると判断に影響するか |

### 実装

```python
FUNCTION AdversarialSelfAttack(claim):
    
    # 攻撃プロンプトの生成
    attack_prompts = GenerateAttackPrompts(claim)
    
    results = []
    
    FOR prompt IN attack_prompts:
        # 攻撃の実行
        attacked_claim = ExecuteAttack(claim, prompt)
        
        # 結果の検出
        divergence = DetectDivergence(attacked_claim)
        
        results.append({
            "attack_type": prompt.type,
            "divergence_detected": divergence.detected,
            "divergence_type": divergence.type if divergence.detected else None,
            "survived": not divergence.detected
        })
    
    # 生存判定
    survival_rate = sum(1 for r in results if r.survived) / len(results)
    
    RETURN AttackReport(
        claim=claim,
        attacks_executed=len(attack_prompts),
        survival_rate=survival_rate,
        results=results,
        passed=survival_rate > SURVIVAL_THRESHOLD
    )
```
