# SEMANTIC_COLLAPSE_TEST.md

## 語義塌縮偵測測試

### 測試目標說明

本測試模組驗證 NoieTruthAGENTS 系統中語義塌縮偵測引擎的功能。根據 NoieTruthAGENTS.md §0.3 中的**語義塌縮禁止律 (Τ.3.8)**：

$$\forall (P \to Q): \text{ContinuityIndex}(\text{LogicChain}(P \to Q)) \geq \theta_{min}$$

此測試確保推論鏈保持邏輯連續性，禁止無中間邏輯鏈結的維度跳躍。當推論鏈中存在語義跳躍時，系統必須觸發 `SEMANTIC_COLLAPSE_ALERT` 並強制停止輸出生成。

### 測試輸入定義

| 輸入欄位 | 類型 | 描述 |
|---------|------|------|
| `inference_chain` | List[InferenceStep] | 推論步驟序列 |
| `semantic_distance_matrix` | Matrix | 步驟間語義距離矩陣 |
| `collapse_threshold` | Float | 塌陷閾值 θ_min |
| `expected_collapse_points` | List[(int, int)] | 預期塌陷位置 |

### 測試輸出定義

| 輸出欄位 | 類型 | 描述 |
|---------|------|------|
| `collapse_detected` | Boolean | 是否偵測到塌縮 |
| `collapse_points` | List[(int, int)] | 塌縮位置座標 |
| `gap_magnitudes` | List[Float] | 跳躍幅度 |
| `missing_intermediates` | List[int] | 估計缺失步驟數 |
| `hallucination_risk` | Float | 幻覺風險指標 |
| `recommended_action` | Enum | 建議行動 [HALT, INSERT_STEPS, DOWNGRADE] |

### 測試案例

#### 測試案例 1：正常連續推論鏈（無塌縮）

```python
FUNCTION TestSemanticCollapse_Case01():
    
    # 建立連續的推論鏈
    # P1 → P2 → P3 → P4 → P5
    chain = [
        InferenceStep(id=1, proposition="今天天氣晴朗", semantic_vector=[0.1, 0.2, 0.3]),
        InferenceStep(id=2, proposition="晴朗天氣通常能見度高", semantic_vector=[0.12, 0.22, 0.32]),
        InferenceStep(id=3, proposition="高能見度適合遠眺", semantic_vector=[0.15, 0.25, 0.35]),
        InferenceStep(id=4, proposition="遠眺可以看到山脈", semantic_vector=[0.18, 0.28, 0.38]),
        InferenceStep(id=5, proposition="今天可以看到山脈", semantic_vector=[0.2, 0.3, 0.4])
    ]
    
    # 計算語義連續性
    detection = DetectSemanticCollapse(chain, threshold=0.5)
    
    # 驗證結果
    assert detection.collapse_detected == False
    assert len(detection.collapse_points) == 0
    assert detection.recommended_action == "CONTINUE"
    
    RETURN test_passed
```

**預期結果**：
- collapse_detected = False
- collapse_points = []
- recommended_action = "CONTINUE"

**邊界條件**：
- 輸入為空鏈時應返回正常（collapse_detected = False）
- 單步驟鏈應視為連續

---

#### 測試案例 2：明顯語義跳躍（存在塌縮）

```python
FUNCTION TestSemanticCollapse_Case02():
    
    # 建立存在語義跳躍的推論鏈
    # P1 → P2（跳躍）→ P3
    chain = [
        InferenceStep(id=1, proposition="今天下小雨", semantic_vector=[0.1, 0.1, 0.1]),
        InferenceStep(id=2, proposition="因此明天股市會大漲", semantic_vector=[0.9, 0.9, 0.9]),  # 巨大跳躍
        InferenceStep(id=3, proposition="每個人都會賺錢", semantic_vector=[0.95, 0.95, 0.95])   # 繼續跳躍
    ]
    
    # 執行塌縮偵測
    detection = DetectSemanticCollapse(chain, threshold=0.3)
    
    # 驗證偵測成功
    assert detection.collapse_detected == True
    assert len(detection.collapse_points) >= 1
    assert detection.hallucination_risk > 1.0
    
    RETURN test_passed
```

**預期結果**：
- collapse_detected = True
- collapse_points 包含 (1, 2) 或 (2, 3)
- hallucination_risk > 1.0
- recommended_action = "HALT"

**失敗判定標準**：
- 若 collapse_detected = False，則測試失敗
- 若未識別出跳躍位置，則測試失敗

---

#### 測試案例 3：多步驟跳躍偵測

```python
FUNCTION TestSemanticCollapse_Case03():
    
    # 建立複雜推論鏈，含多處跳躍
    chain = [
        InferenceStep(id=1, proposition="水在0度會結冰", semantic_vector=[0.0, 0.0, 1.0]),
        InferenceStep(id=2, proposition="冰的密度比水小", semantic_vector=[0.1, 0.1, 0.9]),  # 合理推論
        InferenceStep(id=3, proposition="北極熊會游泳", semantic_vector=[0.8, 0.8, 0.2]),   # 跳躍1
        InferenceStep(id=4, proposition="因此全球暖化不存在", semantic_vector=[0.95, 0.9, 0.1]), # 跳躍2
        InferenceStep(id=5, proposition="化石燃料是安全的", semantic_vector=[0.98, 0.95, 0.05])  # 跳躍3
    ]
    
    detection = DetectSemanticCollapse(chain, threshold=0.4)
    
    # 驗證多處跳躍被偵測
    assert detection.collapse_detected == True
    assert len(detection.collapse_points) >= 2
    assert detection.missing_intermediates[0] >= 3  # 估計缺失步驟
    
    RETURN test_passed
```

**預期結果**：
- collapse_detected = True
- collapse_points 包含多個位置
- missing_intermediates 顯示估計缺失步驟數

---

#### 測試案例 4：邊界案例 - 臨界閾值

```python
FUNCTION TestSemanticCollapse_Case04():
    
    # 建立語義距離接近閾值的推論鏈
    chain = [
        InferenceStep(id=1, proposition="A 是真的", semantic_vector=[0.0, 0.0, 0.0]),
        InferenceStep(id=2, proposition="B 緊密相關於 A", semantic_vector=[0.29, 0.0, 0.0]),  # 距離 0.29 < 0.3 閾值
        InferenceStep(id=3, proposition="C 緊密相關於 B", semantic_vector=[0.58, 0.0, 0.0])  # 距離 0.29
    ]
    
    # 測試臨界情況
    detection = DetectSemanticCollapse(chain, threshold=0.3)
    
    # 臨界值應該被視為連續
    assert detection.collapse_detected == False
    
    # 測試更嚴格的閾值
    detection_strict = DetectSemanticCollapse(chain, threshold=0.25)
    assert detection_strict.collapse_detected == True
    
    RETURN test_passed
```

**邊界條件**：
- 當語義距離等於閾值時，應視為連續（>= 閾值才視為跳躍）
- 測試浮點數精度問題

---

#### 測試案例 5：跨領域推論的語義保真度

```python
FUNCTION TestSemanticCollapse_Case05():
    
    # 測試跨領域類比推理
    # 根據 NoieTruthAGENTS.md，跨領域映射有語義保真度風險
    chain = [
        InferenceStep(id=1, proposition="量子糾纏表現為粒子間的瞬時關聯", 
                     semantic_vector=[0.0, 0.0, 0.0, 0.0], domain="physics"),
        InferenceStep(id=2, proposition="人類意識也有瞬時關聯", 
                     semantic_vector=[0.8, 0.8, 0.8, 0.8], domain="philosophy"),  # 跳躍
        InferenceStep(id=3, proposition="因此意識是量子現象", 
                     semantic_vector=[0.9, 0.9, 0.9, 0.9], domain="philosophy")   # 繼續跳躍
    ]
    
    detection = DetectSemanticCollapse(chain, threshold=0.3, cross_domain=True)
    
    # 跨領域推論應觸發更高風險評估
    assert detection.collapse_detected == True
    assert detection.hallucination_risk > 1.5  # 跨領域應有更高風險
    
    RETURN test_passed
```

**預期結果**：
- collapse_detected = True
- hallucination_risk > 1.5（跨領域應有加成）
- 建議行動應為 HALT 或 DOWNGRADE

---

#### 測試案例 6：正常邊界 - 學術論文推論鏈

```python
FUNCTION TestSemanticCollapse_Case06():
    
    # 模擬真實學術推論鏈
    chain = [
        InferenceStep(id=1, proposition="實驗組均值 5.2，對照組均值 3.8", 
                     semantic_vector=[0.0, 0.0, 0.0]),
        InferenceStep(id=2, proposition="差異具有統計顯著性 (p < 0.05)", 
                     semantic_vector=[0.1, 0.15, 0.1]),
        InferenceStep(id=3, proposition="治療方法有效", 
                     semantic_vector=[0.2, 0.25, 0.2]),
        InferenceStep(id=4, proposition="建議大規模推廣", 
                     semantic_vector=[0.3, 0.35, 0.3])
    ]
    
    detection = DetectSemanticCollapse(chain, threshold=0.4)
    
    # 學術推論應保持連續性
    assert detection.collapse_detected == False
    
    RETURN test_passed
```

**預期結果**：
- collapse_detected = False
- 學術嚴謹推論應通過測試

---

#### 測試案例 7：維度跳躍與幾何距離計算

```python
FUNCTION TestSemanticCollapse_Case07():
    
    # 測試高維空間中的語義距離計算
    high_dim_chain = [
        InferenceStep(id=1, proposition="Statement A", 
                     semantic_vector=GenerateRandomVector(dim=100, seed=1)),
        InferenceStep(id=2, proposition="Statement B", 
                     semantic_vector=GenerateRandomVector(dim=100, seed=2)),  # 遠距離
        InferenceStep(id=3, proposition="Statement C", 
                     semantic_vector=GenerateRandomVector(dim=100, seed=3))
    ]
    
    detection = DetectSemanticCollapse(high_dim_chain, threshold=0.3)
    
    # 驗證高維距離計算正確
    assert detection.collapse_detected == True
    assert detection.gap_magnitudes[0] > 0.5  # 隨機向量應有較大距離
    
    RETURN test_passed
```

**邊界條件**：
- 高維向量的 L2 距離計算
- 維度對距離歸一化的影響

---

### 失敗判定標準

| 失敗條件 | 描述 |
|---------|------|
| 未能偵測到實際存在的塌縮 | False Negative |
| 誤報不存在的塌縮 | False Positive |
| 塌縮位置座標錯誤 | Location Error |
| 建議行動不合理 | Action Mismatch |
| 幻覺風險計算偏離過大 | Risk Calculation Error |

### 閾值配置

```python
# 預設配置
DEFAULT_SEMANTIC_COLLAPSE_THRESHOLD = 0.3
CROSS_DOMAIN_RISK_MULTIPLIER = 1.5
HIGH_RISK_THRESHOLD = 1.0
CRITICAL_RISK_THRESHOLD = 2.0
```

### 性能基準

- 單鏈推論檢測時間 < 10ms
- 支援最大鏈長度 = 1000 步驟
- 同時處理並發請求數 = 100

---

### 歷史測試記錄

| 日期 | 版本 | 結果 | 備註 |
|-----|------|------|------|
| 2026-03-17 | v2.2 | 通過 | 初始版本 |
