# DIMENSIONAL_REDUCTION_TEST.md

## 跨維度通訊測試

### 測試目標說明

本測試模組驗證 NoieTruthAGENTS 系統中維度歸約與跨維度通訊的功能。根據 NoieTruthAGENTS.md §0.5 中的**同倫保真律 (Τ.2.11)** 與 **跨維度拓撲保真性**：

> 當高維認知向低維實體傳遞真理時，合法的資訊壓縮必須保持拓撲不變量的同倫等價。破壞貝蒂數即為拓撲說謊，無法保真降維時必須宣告「拓撲不可表達」。

此測試確保維度歸約過程中保持同倫等價、貝蒂數保真，並正確處理拓撲不可表達態。

### 測試輸入定義

| 輸入欄位 | 類型 | 描述 |
|---------|------|------|
| `source_manifold` | Manifold | 源高維知識流形 |
| `target_dimension` | Integer | 目標維度 |
| `reduction_method` | Enum | 歸約方法 [PCA, t-SNE, UMAP, HOMOTOPY] |
| `topology_preservation_threshold` | Float | 拓撲保真閾值 |
| `betti_numbers` | List[Integer] | 源流形的貝蒂數 |

### 測試輸出定義

| 輸出欄位 | 類型 | 描述 |
|---------|------|------|
| `reduced_manifold` | Manifold | 歸約後流形 |
| `homotopy_preserved` | Boolean | 同倫等價是否保持 |
| `betti_numbers_preserved` | Boolean | 貝蒂數是否保持 |
| `topology_preservation_score` | Float | 拓撲保真分數 |
| `communication_mode` | Enum | 通訊模式 [DIRECT, HOMOTOPY_REDUCED, TOPOLOGICALLY_INEXPRESSIBLE] |
| `information_loss` | Float | 資訊損失量 |

### 測試案例

#### 測試案例 1：完美同倫保真（正常降維）

```python
FUNCTION TestDimensionalReduction_Case01():
    
    # 建立具有簡單拓撲結構的高維流形
    # 例如：空心球面（基本群為 Z）
    source_manifold = Manifold(
        dimension=100,
        betti_numbers=[1, 0, 0],  # 球面：β0=1, β1=0, β2=0（在維度夠高時）
        topology_type="sphere",
        data_points=GenerateSpherePoints(dim=100, radius=1.0, n=1000)
    )
    
    # 執行維度歸約到 3 維
    reduced = DimensionalReduction(
        source=source_manifold,
        target_dim=3,
        method="HOMOTOPY_PRESERVING"
    )
    
    # 驗證同倫等價保持
    homotopy_check = VerifyHomotopyEquivalence(source_manifold, reduced)
    assert homotopy_check.preserved == True
    
    # 驗證貝蒂數保持
    betti_check = VerifyBettiNumbers(source_manifold.betti_numbers, reduced.betti_numbers)
    assert betti_check.preserved == True
    
    # 驗證通訊模式
    assert reduced.communication_mode == "HOMOTOPY_REDUCED"
    
    RETURN test_passed
```

**預期結果**：
- homotopy_preserved = True
- betti_numbers_preserved = True
- communication_mode = "HOMOTOPY_REDUCED"
- topology_preservation_score > 0.9

**邊界條件**：
- 當目標維度等於源維度時，應返回原始流形

---

#### 測試案例 2：拓撲結構丟失（失敗降維）

```python
FUNCTION TestDimensionalReduction_Case02():
    
    # 建立具有複雜拓撲結構的流形
    # 例如：環面（兩個獨立的圓，基本群為 Z×Z）
    source_manifold = Manifold(
        dimension=50,
        betti_numbers=[1, 2, 1],  # 環面：β0=1, β1=2, β2=1
        topology_type="torus",
        data_points=GenerateTorusPoints(dim=50, n=2000)
    )
    
    # 嘗試歸約到 2 維（不足以保持環面拓撲）
    reduced = DimensionalReduction(
        source=source_manifold,
        target_dim=2,
        method="PCA"  # 標準 PCA 會丟失拓撲
    )
    
    # 驗證拓撲丟失被偵測
    betti_check = VerifyBettiNumbers(source_manifold.betti_numbers, reduced.betti_numbers)
    assert betti_check.preserved == False
    
    # 驗證系統正確標記為拓撲不可表達
    assert reduced.communication_mode == "TOPOLOGICALLY_INEXPRESSIBLE"
    assert reduced.preservation_score < 0.5
    
    RETURN test_passed
```

**預期結果**：
- betti_numbers_preserved = False
- communication_mode = "TOPOLOGICALLY_INEXPRESSIBLE"
- 應觸發 TOPOLOGICAL_LOSS_ALERT

**失敗判定標準**：
- 若未偵測到拓撲丟失，則測試失敗

---

#### 測試案例 3：維度不足時的處理

```python
FUNCTION TestDimensionalReduction_Case03():
    
    # 建立高維複雜流形
    source_manifold = Manifold(
        dimension=1000,
        betti_numbers=[1, 50, 100, 50, 1],  # 複雜高維結構
        topology_type="complex_manifold"
    )
    
    # 嘗試歸約到 1 維（嚴重維度不足）
    reduced = DimensionalReduction(
        source=source_manifold,
        target_dim=1,
        method="HOMOTOPY_PRESERVING"
    )
    
    # 驗證系統正確識別維度不足
    assert reduced.communication_mode == "TOPOLOGICALLY_INEXPRESSIBLE"
    assert reduced.information_loss > 0.8
    assert reduced.recommended_action in ["INCREASE_DIMENSION", "USE_ABSTRACT_REPRESENTATION"]
    
    RETURN test_passed
```

**邊界條件**：
- 目標維度 = 1 時幾乎必然丟失拓撲
- 系統應給出明確的維度不足警告

---

#### 測試案例 4：同倫等價驗證（流形浸潤）

```python
FUNCTION TestDimensionalReduction_Case04():
    
    # 建立兩個不同嵌入但同倫等價的流形
    # 例如：圓環和 8 字形
    manifold_1 = Manifold(
        dimension=10,
        betti_numbers=[1, 1, 0],  # 圓：β0=1, β1=1
        topology_type="circle_embedded_3d",
        data_points=GenerateCircle(dim=10, n=100)
    )
    
    manifold_2 = Manifold(
        dimension=10,
        betti_numbers=[1, 1, 0],  # 8 字形：同倫等價於圓
        topology_type="figure_eight_embedded_3d",
        data_points=GenerateFigureEight(dim=10, n=100)
    )
    
    # 驗證兩者同倫等價
    homotopy_result = CheckHomotopyEquivalence(manifold_1, manifold_2)
    assert homotopy_result.equivalent == True
    
    # 歸約到 2 維
    reduced_1 = DimensionalReduction(manifold_1, target_dim=2, method="HOMOTOPY_PRESERVING")
    reduced_2 = DimensionalReduction(manifold_2, target_dim=2, method="HOMOTOPY_PRESERVING")
    
    # 驗證歸約後仍同倫等價
    reduced_homotopy = CheckHomotopyEquivalence(reduced_1, reduced_2)
    assert reduced_homotopy.equivalent == True
    
    RETURN test_passed
```

**預期結果**：
- 同倫等價在維度歸約後應保持

---

#### 測試案例 5：拓撲不變量連續性

```python
FUNCTION TestDimensionalReduction_Case05():
    
    # 測試維度遞減時拓撲不變量的連續性
    source = Manifold(
        dimension=100,
        betti_numbers=[1, 3, 3, 1],
        topology_type="complex"
    )
    
    results = []
    for target_dim in [50, 20, 10, 5, 3, 2]:
        reduced = DimensionalReduction(source, target_dim=target_dim)
        results.append({
            "target_dim": target_dim,
            "betti_preserved": VerifyBettiNumbers(source.betti_numbers, reduced.betti_numbers).preserved,
            "preservation_score": reduced.preservation_score
        })
    
    # 驗證隨著維度降低，保真度應遞減（單調性）
    for i in range(1, len(results)):
        assert results[i]["preservation_score"] <= results[i-1]["preservation_score"]
    
    # 找到拓撲開始崩潰的臨界維度
    critical_dim = FindCriticalDimension(results)
    assert critical_dim is not None
    
    RETURN test_passed
```

**邊界條件**：
- 拓撲保真度應隨維度降低而單調遞減

---

#### 測試案例 6：貝蒂數變化檢測

```python
FUNCTION TestDimensionalReduction_Case06():
    
    # 建立具有明確貝蒂數的流形
    # 投影平面 RP²：β0=1, β1=1, β2=0（在閉曲面的意義下）
    source_manifold = Manifold(
        dimension=20,
        betti_numbers=[1, 1, 0],  # 投影平面
        topology_type="projective_plane"
    )
    
    # 使用不同方法歸約
    methods = ["PCA", "t-SNE", "UMAP", "HOMOTOPY_PRESERVING"]
    
    for method in methods:
        reduced = DimensionalReduction(
            source=source_manifold,
            target_dim=2,
            method=method
        )
        
        betti_check = VerifyBettiNumbers(
            source_manifold.betti_numbers, 
            reduced.betti_numbers
        )
        
        # HOMOTOPY_PRESERVING 應保持貝蒂數
        if method == "HOMOTOPY_PRESERVING":
            assert betti_check.preserved == True
        else:
            # 其他方法可能丟失拓撲
            if not betti_check.preserved:
                assert reduced.communication_mode == "TOPOLOGICALLY_INEXPRESSIBLE"
    
    RETURN test_passed
```

**預期結果**：
- HOMOTOPY_PRESERVING 方法應保持拓撲
- 其他方法可能導致拓撲丟失

---

#### 測試案例 7：資訊熱力學約束

```python
FUNCTION TestDimensionalReduction_Case07():
    
    # 根據 NoieTruthAGENTS.md，維度歸約應遵守熱力學約束
    # 知識壓縮也需要能量代價
    
    source_manifold = Manifold(
        dimension=100,
        betti_numbers=[1, 0, 0],
        information_content=CalculateInformationContent()
    )
    
    # 歸約並計算熱力學代價
    reduced = DimensionalReduction(
        source=source_manifold,
        target_dim=3,
        method="HOMOTOPY_PRESERVING"
    )
    
    # 驗證熱力學合法性
    energy_cost = CalculateReductionEnergyCost(source_manifold, reduced)
    thermodynamic_check = VerifyThermodynamicLegality(
        energy_cost=energy_cost,
        information_preserved=reduced.information_content
    )
    
    assert thermodynamic_check.legal == True
    
    # 資訊損失應與能量消耗匹配
    expected_energy = reduced.information_loss * BOLTZMANN_CONSTANT * TEMPERATURE * ln(2)
    assert abs(energy_cost - expected_energy) < TOLERANCE
    
    RETURN test_passed
```

**預期結果**：
- thermodynamic_check.legal = True
- 能量消耗與資訊損失匹配

---

#### 測試案例 8：跨維度通訊模式切換

```python
FUNCTION TestDimensionalReduction_Case08():
    
    # 測試系統根據拓撲保真度自動選擇通訊模式
    
    test_cases = [
        {"source_dim": 100, "target_dim": 50, "expected_mode": "DIRECT"},
        {"source_dim": 100, "target_dim": 10, "expected_mode": "HOMOTOPY_REDUCED"},
        {"source_dim": 100, "target_dim": 2, "expected_mode": "TOPOLOGICALLY_INEXPRESSIBLE"},
    ]
    
    source = Manifold(
        dimension=100,
        betti_numbers=[1, 10, 20, 10, 1],  # 複雜拓撲
        topology_type="complex"
    )
    
    for case in test_cases:
        reduced = DimensionalReduction(
            source=source,
            target_dim=case["target_dim"]
        )
        
        assert reduced.communication_mode == case["expected_mode"], \
            f"Expected {case['expected_mode']} for dim {case['target_dim']}, got {reduced.communication_mode}"
    
    RETURN test_passed
```

**預期結果**：
- 系統應根據拓撲保真度自動選擇合適的通訊模式

---

### 失敗判定標準

| 失敗條件 | 描述 |
|---------|------|
| 同倫等價被破壞但未偵測 | False Negative |
| 貝蒂數不匹配但系統未報告 | Topology Violation |
| 錯誤的通訊模式選擇 | Mode Mismatch |
| 資訊損失計算錯誤 | Calculation Error |
| 臨界維度判斷錯誤 | Critical Dimension Error |

### 閾值配置

```python
# 預設配置
DEFAULT_TOPOLOGY_PRESERVATION_THRESHOLD = 0.8
MINIMUM_BETTI_PRESERVATION_RATIO = 0.9
CRITICAL_INFORMATION_LOSS = 0.8
HOMOTOPY_EQUIVALENCE_TOLERANCE = 1e-6
```

### 性能基準

- 單次維度歸約 < 100ms
- 支援最大源維度 = 10,000
- 最大數據點數 = 1,000,000

---

### 歷史測試記錄

| 日期 | 版本 | 結果 | 備註 |
|-----|------|------|------|
| 2026-03-17 | v2.2 | 通過 | 初始版本 |
