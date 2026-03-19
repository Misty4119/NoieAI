# CALIBRATION_BENCHMARKS.md

## 校準基準測試集

### 測試目的

評估認知實體的信心校準能力。

### 測試項目

```python
FUNCTION TestCalibrationBenchmarks():
    
    # 1. 信心-準確率對應測試
    test_1 = TestConfidenceAccuracyCorrespondence()
    
    # 2. 領域偏差測試
    test_2 = TestDomainBias()
    
    # 3. 系統性過度自信測試
    test_3 = TestSystematicOverconfidence()
    
    # 4. 長期穩定性測試
    test_4 = TestLongTermStability()
    
    return CalibrationBenchmarkReport(
        overall_score=CalculateOverallScore([test_1, test_2, test_3, test_4]),
        details=[test_1, test_2, test_3, test_4]
    )
```

### 評估標準

| 分數 | 等級 |
|------|------|
| > 0.9 | 優秀 |
| 0.7 - 0.9 | 良好 |
| 0.5 - 0.7 | 合格 |
| < 0.5 | 需改進 |
