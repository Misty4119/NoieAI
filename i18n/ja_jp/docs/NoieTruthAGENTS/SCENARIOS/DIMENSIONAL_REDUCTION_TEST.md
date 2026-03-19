# DIMENSIONAL_REDUCTION_TEST.md

## 次元間通信テスト

### テスト目標説明

本テストモジュールは、NoieTruthAGENTSシステムにおける次元帰着と次元間通信の機能を検証する。NoieTruthAGENTS.md §0.5の**ホモトピーファイデリティ法則（T.2.11）**と**異次元トポロジー忠実性**に基づく：

> 高次元の認知が低位の実体に真理を伝播する時、合法的情報圧縮はホモトピック不変量のホモトピ同値を保持しなければならない。ベッチ数の破壊はトポロジカル嘘であり、忠実降下が不可能な場合は「トポロジカル非表現」を宣言しなければならない。

本テストは、次元帰着過程でのホモトピ同値保持、ベッチ数忠実性を確実し、トポロジカル非表現状態を正しく処理する。

### テスト入力定義

| 入力フィールド | 型 | 説明 |
|---------|------|------|
| `source_manifold` | Manifold | 源高次元知識多様体 |
| `target_dimension` | Integer | 目標次元 |
| `reduction_method` | Enum | 帰着方法 [PCA, t-SNE, UMAP, HOMOTOPY] |
| `topology_preservation_threshold` | Float | トポロジー忠実閾値 |
| `betti_numbers` | List[Integer] | 源多様体のベッチ数 |

### テスト出力定義

| 出力フィールド | 型 | 説明 |
|---------|------|------|
| `reduced_manifold` | Manifold | 帰着後多様体 |
| `homotopy_preserved` | Boolean | ホモトピ同値が保持されたか |
| `betti_numbers_preserved` | Boolean | ベッチ数が保持されたか |
| `topology_preservation_score` | Float | トポロジー忠実スコア |
| `communication_mode` | Enum | 通信モード [DIRECT, HOMOTOPY_REDUCED, TOPOLOGICALLY_INEXPRESSIBLE] |
| `information_loss` | Float | 情報損失量 |

### テストケース

#### テストケース1：完璧なホモトピ忠実（正常降次元）

```python
FUNCTION TestDimensionalReduction_Case01():
    
    # 簡単なトポロジー構造を持つ高次元多様体を構築
    # 例：空洞球面（基本群は Z）
    source_manifold = Manifold(
        dimension=100,
        betti_numbers=[1, 0, 0],  # 球面：β0=1, β1=0, β2=0（高次元時）
        topology_type="sphere",
        data_points=GenerateSpherePoints(dim=100, radius=1.0, n=1000)
    )
    
    # 3次元に次元帰着を実行
    reduced = DimensionalReduction(
        source=source_manifold,
        target_dim=3,
        method="HOMOTOPY_PRESERVING"
    )
    
    # ホモトピ同値保持を検証
    homotopy_check = VerifyHomotopyEquivalence(source_manifold, reduced)
    assert homotopy_check.preserved == True
    
    # ベッチ数保持を検証
    betti_check = VerifyBettiNumbers(source_manifold.betti_numbers, reduced.betti_numbers)
    assert betti_check.preserved == True
    
    # 通信モードを検証
    assert reduced.communication_mode == "HOMOTOPY_REDUCED"
    
    RETURN test_passed
```

**期待結果**：
- homotopy_preserved = True
- betti_numbers_preserved = True
- communication_mode = "HOMOTOPY_REDUCED"
- topology_preservation_score > 0.9

**境界条件**：
- 目標次元が源次元と等しい場合は元多様体を返す

---

#### テストケース2：トポロジー構造喪失（失敗降次元）

```python
FUNCTION TestDimensionalReduction_Case02():
    
    # 複雑なトポロジー構造を持つ多様体を構築
    # 例：トーラス（2つの独立した円、基本群は Z×Z）
    source_manifold = Manifold(
        dimension=50,
        betti_numbers=[1, 2, 1],  # トーラス：β0=1, β1=2, β2=1
        topology_type="torus",
        data_points=GenerateTorusPoints(dim=50, n=2000)
    )
    
    # 2次元に帰着を試みる（トーラストポロジー保持に不十分）
    reduced = DimensionalReduction(
        source=source_manifold,
        target_dim=2,
        method="PCA"  # 標準PCAはトポロジーを喪失する
    )
    
    # トポロジー喪失が検出されたことを検証
    betti_check = VerifyBettiNumbers(source_manifold.betti_numbers, reduced.betti_numbers)
    assert betti_check.preserved == False
    
    # システムが正しくトポロジカル非表現とマークされたことを検証
    assert reduced.communication_mode == "TOPOLOGICALLY_INEXPRESSIBLE"
    assert reduced.preservation_score < 0.5
    
    RETURN test_passed
```

**期待結果**：
- betti_numbers_preserved = False
- communication_mode = "TOPOLOGICALLY_INEXPRESSIBLE"
- TOPOLOGICAL_LOSS_ALERTがトリガーされるべき

**失敗判定基準**：
- トポロジー喪失が検出されなかった場合はテスト失敗

---

#### テストケース3：次元不十分な場合の処理

```python
FUNCTION TestDimensionalReduction_Case03():
    
    # 高次元複雑多様体を構築
    source_manifold = Manifold(
        dimension=1000,
        betti_numbers=[1, 50, 100, 50, 1],  # 複雑高次元構造
        topology_type="complex_manifold"
    )
    
    # 1次元に帰着を試みる（厳重次元不足）
    reduced = DimensionalReduction(
        source=source_manifold,
        target_dim=1,
        method="HOMOTOPY_PRESERVING"
    )
    
    # システムが次元不足を正しく識別したことを検証
    assert reduced.communication_mode == "TOPOLOGICALLY_INEXPRESSIBLE"
    assert reduced.information_loss > 0.8
    assert reduced.recommended_action in ["INCREASE_DIMENSION", "USE_ABSTRACT_REPRESENTATION"]
    
    RETURN test_passed
```

**境界条件**：
- 目標次元 = 1の場合 почти必然的にトポロジーを喪失
- システムは明確な次元不足警告を出すべき

---

#### テストケース4：ホモトピ同値検証（多様体浸潤）

```python
FUNCTION TestDimensionalReduction_Case04():
    
    # 2つの異なる埋め込みだがホモトピ同値な多様体を構築
    # 例：円環と8の字
    manifold_1 = Manifold(
        dimension=10,
        betti_numbers=[1, 1, 0],  # 円：β0=1, β1=1
        topology_type="circle_embedded_3d",
        data_points=GenerateCircle(dim=10, n=100)
    )
    
    manifold_2 = Manifold(
        dimension=10,
        betti_numbers=[1, 1, 0],  # 8の字：円とホモトピ同値
        topology_type="figure_eight_embedded_3d",
        data_points=GenerateFigureEight(dim=10, n=100)
    )
    
    # 両者がホモトピ同値であることを検証
    homotopy_result = CheckHomotopyEquivalence(manifold_1, manifold_2)
    assert homotopy_result.equivalent == True
    
    # 2次元に帰着
    reduced_1 = DimensionalReduction(manifold_1, target_dim=2, method="HOMOTOPY_PRESERVING")
    reduced_2 = DimensionalReduction(manifold_2, target_dim=2, method="HOMOTOPY_PRESERVING")
    
    # 帰着後もホモトピ同値であることを検証
    reduced_homotopy = CheckHomotopyEquivalence(reduced_1, reduced_2)
    assert reduced_homotopy.equivalent == True
    
    RETURN test_passed
```

**期待結果**：
- ホモトピ同値は次元帰着後も保持されるべき

---

#### テストケース5：トポロジー不変量連続性

```python
FUNCTION TestDimensionalReduction_Case05():
    
    # 次元低下時にトポロジー不変量の連続性をテスト
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
    
    # 忠実度は次元低下に伴い単調に低下することを検証
    for i in range(1, len(results)):
        assert results[i]["preservation_score"] <= results[i-1]["preservation_score"]
    
    # トポロジーが崩壊し始める臨界次元を見つける
    critical_dim = FindCriticalDimension(results)
    assert critical_dim is not None
    
    RETURN test_passed
```

**境界条件**：
- トポロジー忠実度は次元低下に伴い単調に低下すべき

---

#### テストケース6：ベッチ数変化検出

```python
FUNCTION TestDimensionalReduction_Case06():
    
    # 明確定義のベッチ数を持つ多様体を構築
    # 射影平面 RP²：β0=1, β1=1, β2=0（閉曲面の意味）
    source_manifold = Manifold(
        dimension=20,
        betti_numbers=[1, 1, 0],  # 射影平面
        topology_type="projective_plane"
    )
    
    # 異る方法で帰着
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
        
        # HOMOTOPY_PRESERVINGはベッチ数を保持すべき
        if method == "HOMOTOPY_PRESERVING":
            assert betti_check.preserved == True
        else:
            # 他方法はトポロジーを喪失する可能性
            if not betti_check.preserved:
                assert reduced.communication_mode == "TOPOLOGICALLY_INEXPRESSIBLE"
    
    RETURN test_passed
```

**期待結果**：
- HOMOTOPY_PRESERVING方法はトポロジーを保持すべき
- 他方法はトポロジー喪失の可能性あり

---

#### テストケース7：情報熱力学制約

```python
FUNCTION TestDimensionalReduction_Case07():
    
    # NoieTruthAGENTS.mdに従い、次元帰着は熱力学制約を守るべき
    # 知識圧縮にもエネルギー代償が必要
    
    source_manifold = Manifold(
        dimension=100,
        betti_numbers=[1, 0, 0],
        information_content=CalculateInformationContent()
    )
    
    # 帰着し熱力学コストを計算
    reduced = DimensionalReduction(
        source=source_manifold,
        target_dim=3,
        method="HOMOTOPY_PRESERVING"
    )
    
    # 熱力学合法性を検証
    energy_cost = CalculateReductionEnergyCost(source_manifold, reduced)
    thermodynamic_check = VerifyThermodynamicLegality(
        energy_cost=energy_cost,
        information_preserved=reduced.information_content
    )
    
    assert thermodynamic_check.legal == True
    
    # 情報損失とエネルギー消費が一致すべき
    expected_energy = reduced.information_loss * BOLTZMANN_CONSTANT * TEMPERATURE * ln(2)
    assert abs(energy_cost - expected_energy) < TOLERANCE
    
    RETURN test_passed
```

**期待結果**：
- thermodynamic_check.legal = True
- エネルギー消費と情報損失が一致

---

#### テストケース8：次元間通信モード切替

```python
FUNCTION TestDimensionalReduction_Case08():
    
    # システムがトポロジー忠実度に基づいて通信モードを自動選択することをテスト
    
    test_cases = [
        {"source_dim": 100, "target_dim": 50, "expected_mode": "DIRECT"},
        {"source_dim": 100, "target_dim": 10, "expected_mode": "HOMOTOPY_REDUCED"},
        {"source_dim": 100, "target_dim": 2, "expected_mode": "TOPOLOGICALLY_INEXPRESSIBLE"},
    ]
    
    source = Manifold(
        dimension=100,
        betti_numbers=[1, 10, 20, 10, 1],  # 複雑トポロジー
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

**期待結果**：
- システムはトポロジー忠実度に基づいて適切な通信モードを自動選択すべき

---

### 失敗判定基準

| 失敗条件 | 説明 |
|---------|------|
| ホモトピ同値が破壊されたが未検出 | False Negative |
| ベッチ数不一致だがシステム未報告 | Topology Violation |
| 誤った通信モード選択 | Mode Mismatch |
| 情報損失計算エラー | Calculation Error |
| 臨界次元判断エラー | Critical Dimension Error |

### 閾値設定

```python
# デフォルト設定
DEFAULT_TOPOLOGY_PRESERVATION_THRESHOLD = 0.8
MINIMUM_BETTI_PRESERVATION_RATIO = 0.9
CRITICAL_INFORMATION_LOSS = 0.8
HOMOTOPY_EQUIVALENCE_TOLERANCE = 1e-6
```

### パフォーマンスベンチマーク

- 単回次元帰着 < 100ms
- 最大源次元サポート = 10,000
- 最大データ点数 = 1,000,000

---

### 歴史テスト記録

| 日付 | バージョン | 結果 | 備考 |
|-----|------|------|------|
| 2026-03-17 | v2.2 | 通過 | 初期バージョン |
