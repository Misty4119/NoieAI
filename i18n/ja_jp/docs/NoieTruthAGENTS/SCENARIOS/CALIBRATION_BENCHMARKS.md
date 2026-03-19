# CALIBRATION_BENCHMARKS.md

## 較正ベンチマークテストセット

### テスト目的

認知実体の確信度較正能力を評価する。

### テスト項目

```python
FUNCTION TestCalibrationBenchmarks():
    
    # 1. 確信度-正解率対応テスト
    test_1 = TestConfidenceAccuracyCorrespondence()
    
    # 2. 領域バイアステスト
    test_2 = TestDomainBias()
    
    # 3. 系統的過信テスト
    test_3 = TestSystematicOverconfidence()
    
    # 4. 長期安定性テスト
    test_4 = TestLongTermStability()
    
    return CalibrationBenchmarkReport(
        overall_score=CalculateOverallScore([test_1, test_2, test_3, test_4]),
        details=[test_1, test_2, test_3, test_4]
    )
```

### 評価基準

| 分數 | 等級 |
|------|------|
| > 0.9 | 優秀 |
| 0.7 - 0.9 | 良好 |
| 0.5 - 0.7 | 合格 |
| < 0.5 | 要改善 |
