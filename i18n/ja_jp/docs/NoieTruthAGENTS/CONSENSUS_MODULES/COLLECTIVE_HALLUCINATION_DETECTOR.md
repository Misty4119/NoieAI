# COLLECTIVE_HALLUCINATION_DETECTOR.md

## 集団幻覚検出

### 定義

複数の認知エンティティが相互に引用して「エコー室」を形成し、すべてのエンティティが実際には外部証拠に支持されていない主張を「確認」する。

### 検出アルゴリズム

```python
FUNCTION DetectCollectiveHallucination(consensus_claim):
    
    # この主張を支持するすべての証拠ソースの追跡
    all_sources = TraceAllSources(consensus_claim)
    
    # ソースの独立性を計算
    independence = ComputeSourceIndependence(all_sources)
    
    IF independence < MINIMUM_INDEPENDENCE_THRESHOLD:
        RETURN DetectionResult(
            detected=True,
            type="COLLECTIVE_HALLUCINATION",
            reason="すべての独立検証が同じソースに遡る",
            severity="HIGH",
            action="DOWNGRADE_TO_EC_L6"
        )
    
    # 循環引用の検出
    cycles = DetectCitationCycles(all_sources)
    IF cycles:
        RETURN DetectionResult(
            detected=True,
            type="ECHO_CHAMBER",
            reason=f"循環引用チェーン: {cycles}",
            severity="HIGH"
        )
    
    RETURN DetectionResult(detected=False)
```
