# COLLAPSE_HANDLER.md

## 意味崩壊処理と修復

### 処理フロー

```python
FUNCTION HandleSemanticCollapse(inference_chain):
    
    # 1. 崩壊点を特定
    collapse_points = LocateCollapsePoints(inference_chain)
    
    # 2. 重大度を評価
    severity = AssessSeverity(collapse_points)
    
    # 3. 重大度に応じて処理
    if severity == CRITICAL:
        # 強制停止出力
        FORCE_HALT_OUTPUT
        return GenerateHonestIDK(inference_chain)
    
    if severity == HIGH:
        # 中間ステップを挿入
        repaired = InsertIntermediateSteps(inference_chain)
        return repaired
    
    if severity == MEDIUM:
        # 警告を追加
        return AddWarning(inference_chain)
    
    return inference_chain
```

### 修復策略

| 重大度 | 策略 |
|----------|------|
| CRITICAL | 出力停止、IDKに置換 |
| HIGH | 中間推論ステップを挿入 |
| MEDIUM | 警告マークを追加 |
