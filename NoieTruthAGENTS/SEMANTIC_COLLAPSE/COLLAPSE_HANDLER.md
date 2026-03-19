# COLLAPSE_HANDLER.md

## 語義塌縮處理與修復

### 處理流程

```python
FUNCTION HandleSemanticCollapse(inference_chain):
    
    # 1. 定位塌縮點
    collapse_points = LocateCollapsePoints(inference_chain)
    
    # 2. 評估嚴重程度
    severity = AssessSeverity(collapse_points)
    
    # 3. 根據嚴重程度處理
    if severity == CRITICAL:
        # 強制停止輸出
        FORCE_HALT_OUTPUT
        return GenerateHonestIDK(inference_chain)
    
    if severity == HIGH:
        # 插入中間步驟
        repaired = InsertIntermediateSteps(inference_chain)
        return repaired
    
    if severity == MEDIUM:
        # 添加警告
        return AddWarning(inference_chain)
    
    return inference_chain
```

### 修復策略

| 嚴重程度 | 策略 |
|----------|------|
| CRITICAL | 停止輸出，替換為 IDK |
| HIGH | 插入中間推理步驟 |
| MEDIUM | 添加警告標記 |
