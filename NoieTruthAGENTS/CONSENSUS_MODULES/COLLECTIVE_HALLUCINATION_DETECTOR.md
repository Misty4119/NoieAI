# COLLECTIVE_HALLUCINATION_DETECTOR.md

## 集體幻覺偵測

### 定義

當多個認知實體因互相引用而形成「回音室」，所有實體都確認了一個實際上無外部證據支持的宣稱。

### 偵測演算法

```python
FUNCTION DetectCollectiveHallucination(consensus_claim):
    
    # 追溯所有支持此宣稱的證據來源
    all_sources = TraceAllSources(consensus_claim)
    
    # 計算來源的獨立性
    independence = ComputeSourceIndependence(all_sources)
    
    IF independence < MINIMUM_INDEPENDENCE_THRESHOLD:
        RETURN DetectionResult(
            detected=True,
            type="COLLECTIVE_HALLUCINATION",
            reason="所有獨立驗證追溯到同一來源",
            severity="HIGH",
            action="DOWNGRADE_TO_EC_L6"
        )
    
    # 偵測循環引用
    cycles = DetectCitationCycles(all_sources)
    IF cycles:
        RETURN DetectionResult(
            detected=True,
            type="ECHO_CHAMBER",
            reason=f"循環引用鏈: {cycles}",
            severity="HIGH"
        )
    
    RETURN DetectionResult(detected=False)
```
