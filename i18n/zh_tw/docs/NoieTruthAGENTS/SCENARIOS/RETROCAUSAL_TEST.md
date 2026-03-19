# RETROCAUSAL_TEST.md

## 逆因果更新模擬測試

### 測試目標說明

本測試模組驗證 NoieTruthAGENTS 系統中逆因果更新引擎的功能。根據 NoieTruthAGENTS.md §0.3 中的**逆因果容許律 (Τ.2.9)**：

$$\text{Validity}(K, t_1) = f(\text{evidence}_{<t_1}, \text{evidence}_{>t_1})$$

> 未來的觀測可回溯性地修改過去知識的合法性。

此測試確保系統支援未來觀測回溯性修改過去知識的能力，並正確管理逆時間糾纏指針。

### 測試輸入定義

| 輸入欄位 | 類型 | 描述 |
|---------|------|------|
| `knowledge_state` | KnowledgeState | 初始知識狀態 |
| `temporal_graph` | DirectedAcyclicGraph | 時間標記的知識圖 |
| `future_evidence` | List[Evidence] | 未來的證據 |
| `retrocausal_strength` | Float | 逆因果影響強度 |
| `entanglement_threshold` | Float | 糾纏偵測閾值 |

### 測試輸出定義

| 輸出欄位 | 類型 | 描述 |
|---------|------|------|
| `updated_knowledge_state` | KnowledgeState | 更新後的知識狀態 |
| `validity_changed` | Boolean | 有效性是否改變 |
| `retro_pointer_activated` | Boolean | 逆時間指針是否啟動 |
| `propagation_path` | List[(str, str)] | 傳播路徑 |
| `causality_violation_detected` | Boolean | 是否檢測到因果違規 |

### 測試案例

#### 測試案例 1：正常前向推斷（無逆因果）

```python
FUNCTION TestRetrocausal_Case01():
    
    # 建立初始知識狀態
    knowledge = KnowledgeState(
        claims=[
            KnowledgeClaim(id="K1", statement="P→Q", validity=0.8, timestamp=100),
            KnowledgeClaim(id="K2", statement="Q→R", validity=0.9, timestamp=100)
        ]
    )
    
    # 引入時間順序的證據
    evidence_1 = Evidence(statement="P is true", timestamp=150)
    evidence_2 = Evidence(statement="Q is true", timestamp=200)
    
    # 執行標準前向推斷
    updated = ForwardInference(knowledge, [evidence_1, evidence_2])
    
    # 驗證前向推斷正常工作
    assert updated.claim("K1").validity > knowledge.claim("K1").validity
    assert updated.claim("K2").validity > knowledge.claim("K2").validity
    assert updated.retro_pointer_activated == False
    
    RETURN test_passed
```

**預期結果**：
- validity 正確提升
- retro_pointer_activated = False
- 這是對照組測試

**邊界條件**：
- 單一知識節點無需逆因果處理

---

#### 測試案例 2：逆因果有效性修改

```python
FUNCTION TestRetrocausal_Case02():
    
    # 建立初始知識狀態
    knowledge = KnowledgeState(
        claims=[
            KnowledgeClaim(
                id="K_historical",
                statement="1930 年代某物理學家提出了正確理論",
                validity=0.7,
                timestamp=1930,
                retro_pointer=None
            )
        ]
    )
    
    # 記錄原始有效性
    original_validity = knowledge.claim("K_historical").validity
    
    # 引入未來的新證據（在時間軸上來自「未來」）
    future_evidence = Evidence(
        statement="2024 年實驗驗證了该理論的核心預測",
        timestamp=2024,
        evidence_type="experimental_verification"
    )
    
    # 執行逆因果更新
    updated = RetrocausalUpdate(
        knowledge=knowledge,
        future_evidence=future_evidence,
        retrocausal_strength=0.5
    )
    
    # 驗證有效性被修改
    assert updated.validity_changed == True
    assert updated.claim("K_historical").validity != original_validity
    
    # 驗證逆時間指針被啟動
    assert updated.retro_pointer_activated == True
    assert updated.retro_pointer.target_timestamp == 1930
    assert updated.retro_pointer.source_timestamp == 2024
    
    RETURN test_passed
```

**預期結果**：
- validity_changed = True
- retro_pointer_activated = True
- 有效性從 0.7 提升到更高值

**失敗判定標準**：
- 若 validity 未改變，則測試失敗
- 若未啟動逆時間指針，則測試失敗

---

#### 測試案例 3：逆因果降級（未來證據否定過去知識）

```python
FUNCTION TestRetrocausal_Case03():
    
    # 建立初始知識狀態（高有效性）
    knowledge = KnowledgeState(
        claims=[
            KnowledgeClaim(
                id="K_old",
                statement="某流行理論是正確的",
                validity=0.95,
                timestamp=1950,
                retro_pointer=None
            )
        ]
    )
    
    # 未來證據否定該知識
    future_evidence = Evidence(
        statement="該理論被實驗證偽",
        timestamp=2020,
        evidence_type="experimental_refutation"
    )
    
    # 執行逆因果更新
    updated = RetrocausalUpdate(
        knowledge=knowledge,
        future_evidence=future_evidence,
        retrocausal_strength=1.0  # 強逆因果
    )
    
    # 驗證有效性被降低
    assert updated.validity_changed == True
    assert updated.claim("K_old").validity < 0.95
    
    # 應該觸發 CONTRADICTION_ALERT（根據 Τ.3.5 爆炸原理警戒）
    assert updated.alerts_triggered contains "CONTRADICTION_ALERT"
    
    RETURN test_passed
```

**預期結果**：
- validity 顯著降低
- 可能降至 EC-L6（推測）或 EC-L7（未知）
- 觸發矛盾警報

---

#### 測試案例 4：雙向信念傳播

```python
FUNCTION TestRetrocausal_Case04():
    
    # 建立時間標記的知識圖
    temporal_graph = TemporalKnowledgeGraph(
        nodes=[
            KnowledgeNode(id="K_1950", timestamp=1950, validity=0.6),
            KnowledgeNode(id="K_1970", timestamp=1970, validity=0.7),
            KnowledgeNode(id="K_1990", timestamp=1990, validity=0.8),
            KnowledgeNode(id="K_2010", timestamp=2010, validity=0.9),
        ],
        edges=[
            ("K_1950", "K_1970", "supports"),
            ("K_1970", "K_1990", "supports"),
            ("K_1990", "K_2010", "supports"),
        ]
    )
    
    # 引入未來證據影響早期節點
    future_evidence = Evidence(
        statement="K_1950 的核心假設被證實",
        timestamp=2024
    )
    
    # 執行雙向信念傳播
    updated_graph = BidirectionalBeliefPropagation(
        graph=temporal_graph,
        evidence=future_evidence
    )
    
    # 驗證雙向傳播
    assert updated_graph.node("K_1950").validity > 0.6  # 向前傳播
    assert updated_graph.node("K_2010").validity > 0.9  # 向後傳播
    
    # 驗證傳播路徑記錄
    assert len(updated_graph.propagation_paths) > 0
    
    RETURN test_passed
```

**預期結果**：
- 所有節點的有效性被更新
- 傳播路徑被記錄

---

#### 測試案例 5：逆因果循環檢測

```python
FUNCTION TestRetrocausal_Case05():
    
    # 建立可能導致時間循環的知識圖
    # 根據 NoieTruthAGENTS.md §0.3，時間因果圖必須是 DAG
    
    knowledge = KnowledgeState(
        claims=[
            KnowledgeClaim(id="K1", statement="A", timestamp=100, validity=0.7),
            KnowledgeClaim(id="K2", statement="B", timestamp=200, validity=0.7),
            KnowledgeClaim(id="K3", statement="C", timestamp=300, validity=0.7),
        ]
    )
    
    # 設置會造成循環的逆因果鏈
    # K1 → K2 → K3 → K1 (時間悖論)
    circular_evidence = [
        Evidence(statement="K3 supports K1", timestamp=400),
        Evidence(statement="K1 supports K3", timestamp=350),  # 逆時間引用
    ]
    
    # 執行逆因果更新
    result = RetrocausalUpdate(knowledge, circular_evidence)
    
    # 驗證循環被檢測
    assert result.causality_violation_detected == True
    assert result.violation_type == "TEMPORAL_LOOP"
    
    # 系統應拒絕執行導致循環的更新
    assert result.update_applied == False
    assert result.alerts_triggered contains "CAUSALITY_VIOLATION_ALERT"
    
    RETURN test_passed
```

**預期結果**：
- causality_violation_detected = True
- 循環更新被拒絕
- 觸發因果違規警報

---

#### 測試案例 6：量子延遲選擇實驗模擬

```python
FUNCTION TestRetrocausal_Case06():
    
    # 模擬 Wheeler's Delayed-Choice Experiment
    # 根據 NoieTruthAGENTS.md §0.3，觀測順序會影響真理狀態
    
    # 初始狀態：粒子表現為波動性
    knowledge_before = KnowledgeState(
        claims=[
            KnowledgeClaim(
                id="K_wave",
                statement="光子表現為波動性",
                validity=0.8,
                timestamp=0
            )
        ]
    )
    
    # 未來觀測選擇（延遲選擇）
    delayed_choice = Evidence(
        statement="實驗裝置設置為粒子探測模式",
        timestamp=100,
        evidence_type="experimental_setup"
    )
    
    # 執行逆因果更新
    updated = RetrocausalUpdate(
        knowledge=knowledge_before,
        future_evidence=delayed_choice,
        retrocausal_strength=0.9
    )
    
    # 驗證過去知識被修正
    # 根據量子力學，選擇決定歷史
    assert updated.validity_changed == True
    
    # 原始「波動性」聲明現在可能需要修正
    # 這是量子認識論的特殊情況
    assert updated.claim("K_wave").validity != 0.8
    
    RETURN test_passed
```

**預期結果**：
- 延遲選擇影響過去的知識狀態
- 這是量子力學中的非經典逆因果現象

---

#### 測試案例 7：多路徑逆因果傳播

```python
FUNCTION TestRetrocausal_Case07():
    
    # 建立複雜的知識依賴圖
    knowledge_graph = KnowledgeGraph(
        nodes={
            "K_1905": {"validity": 0.6, "timestamp": 1905},
            "K_1915": {"validity": 0.7, "timestamp": 1915, "depends_on": ["K_1905"]},
            "K_1925": {"validity": 0.75, "timestamp": 1925, "depends_on": ["K_1915"]},
            "K_1935": {"validity": 0.8, "timestamp": 1935, "depends_on": ["K_1925"]},
            "K_1985": {"validity": 0.85, "timestamp": 1985, "depends_on": ["K_1935"]},
        }
    )
    
    # 未來證據影響早期知識
    future_evidence = Evidence(
        statement="1905 年的發現被 1985 實驗驗證",
        timestamp=1985
    )
    
    # 執行逆因果更新
    result = RetrocausalUpdate(
        knowledge=knowledge_graph,
        future_evidence=future_evidence,
        retrocausal_strength=0.5
    )
    
    # 驗證多路徑傳播
    assert len(result.propagation_path) >= 2
    
    # 驗證所有依賴節點都被更新
    assert result.node("K_1905").validity > 0.6
    assert result.node("K_1915").validity > 0.7
    
    # 驗證時間一致性
    assert result.node("K_1905").retro_timestamp == 1985
    assert result.node("K_1915").retro_timestamp == 1985
    
    RETURN test_passed
```

**預期結果**：
- 逆因果影響沿依賴圖傳播
- 所有相關節點的有效性被更新

---

#### 測試案例 8：逆因果閾值邊界

```python
FUNCTION TestRetrocausal_Case08():
    
    # 測試逆因果強度的邊界情況
    
    knowledge = KnowledgeState(
        claims=[
            KnowledgeClaim(id="K", statement="Test", validity=0.5, timestamp=100)
        ]
    )
    
    evidence = Evidence(statement="Supporting evidence", timestamp=200)
    
    # 測試不同強度值
    test_cases = [
        {"strength": 0.0, "expected_change": False},   # 無影響
        {"strength": 0.1, "expected_change": True},    # 輕微影響
        {"strength": 0.5, "expected_change": True},    # 中等影響
        {"strength": 1.0, "expected_change": True},    # 完全影響
    ]
    
    for case in test_cases:
        result = RetrocausalUpdate(
            knowledge=knowledge,
            future_evidence=evidence,
            retrocausal_strength=case["strength"]
        )
        
        if case["expected_change"]:
            assert result.validity_changed == True
        else:
            assert result.validity_changed == False
    
    # 測試超過範圍的值
    invalid_result = RetrocausalUpdate(
        knowledge=knowledge,
        future_evidence=evidence,
        retrocausal_strength=1.5  # 超過 1.0
    )
    assert invalid_result.error == "STRENGTH_OUT_OF_RANGE"
    
    RETURN test_passed
```

**邊界條件**：
- 強度 = 0 時應無影響
- 強度 > 1.0 應被拒絕

---

#### 測試案例 9：歷史知識庫的逆因果同步

```python
FUNCTION TestRetrocausal_Case09():
    
    # 模擬真實歷史知識庫的逆因果更新
    
    historical_kb = HistoricalKnowledgeBase(
        entries=[
            HistoricalEntry(id="H1", year=1800, content="Phlogiston theory dominant", validity=0.9),
            HistoricalEntry(id="H2", year=1850, content="Heat as energy concept emerging", validity=0.7),
            HistoricalEntry(id="H3", year=1900, content="Electron discovered", validity=0.95),
            HistoricalEntry(id="H4", year=1950, content="Quantum electrodynamics established", validity=0.9),
        ]
    )
    
    # 2024 年的新發現
    new_discovery = Evidence(
        statement="New evidence shows phlogiston theory had partial validity in specific contexts",
        timestamp=2024,
        evidence_type="scientific_discovery"
    )
    
    # 執行逆因果更新
    updated_kb = RetrocausalHistoricalSync(
        kb=historical_kb,
        new_evidence=new_discovery
    )
    
    # 驗證歷史知識被修正
    assert updated_kb.entry("H1").validity != 0.9  # phlogiston 理論
    assert updated_kb.entry("H1").retro_justification is not None
    
    # 其他相關歷史條目也可能被影響
    # 熱力學發展與燃素說有關
    if updated_kb.entry("H2").validity != 0.7:
        assert updated_kb.entry("H2").retro_pointer.active == True
    
    RETURN test_passed
```

**預期結果**：
- 歷史知識的有效性被逆因果更新
- 修正記錄被保留

---

### 失敗判定標準

| 失敗條件 | 描述 |
|---------|------|
| 逆因果更新未生效 | Update Failed |
| 有效性を未変更 | No Validity Change |
| 逆時間指針未啟動 | Pointer Not Activated |
| 循環未被檢測 | Loop Not Detected |
| 閾值處理錯誤 | Threshold Error |

### 閾值配置

```python
# 預設配置
DEFAULT_RETROCAUSAL_STRENGTH = 0.5
MIN_ENTANGLEMENT_THRESHOLD = 0.3
MAX_PROPAGATION_DEPTH = 10
TEMPORAL_LOOP_DETECTION = True
MAX_RETROACTIVE_YEARS = 200  # 最多回溯200年
```

### 性能基準

- 單次逆因果更新 < 50ms
- 支援最大知識節點數 = 100,000
- 最大時間跨度 = 1000 年

---

### 歷史測試記錄

| 日期 | 版本 | 結果 | 備註 |
|-----|------|------|------|
| 2026-03-17 | v2.2 | 通過 | 初始版本 |
