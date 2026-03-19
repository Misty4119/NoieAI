# SEMANTIC_COLLAPSE_TEST.md

## 意味崩壊検出テスト

### テスト目標説明

本テストモジュールは、NoieTruthAGENTSシステムにおける意味崩壊検出エンジンの機能を検証する。NoieTruthAGENTS.md §0.3の**意味崩壊禁止法則（T.3.8）**に基づく：

$$\forall (P \to Q): \text{ContinuityIndex}(\text{LogicChain}(P \to Q)) \geq \theta_{min}$$

本テストは推論チェーンが論理的連続性を保持すること、論理的チェインのない次元飛躍を禁止することを確実にする。推論チェーンに意味的飛躍が存在する場合、システムは`SEMANTIC_COLLAPSE_ALERT`をトリガーし、出力生成を強制停止しなければならない。

### テスト入力定義

| 入力フィールド | 型 | 説明 |
|---------|------|------|
| `inference_chain` | List[InferenceStep] | 推論ステップシーケンス |
| `semantic_distance_matrix` | Matrix | ステップ間意味距離行列 |
| `collapse_threshold` | Float | 崩壊閾値 θ_min |
| `expected_collapse_points` | List[(int, int)] | 期待崩壊位置 |

### テスト出力定義

| 出力フィールド | 型 | 説明 |
|---------|------|------|
| `collapse_detected` | Boolean | 崩壊が検出されたか |
| `collapse_points` | List[(int, int)] | 崩壊位置座標 |
| `gap_magnitudes` | List[Float] | 飛躍幅度 |
| `missing_intermediates` | List[int] | 推定欠落ステップ数 |
| `hallucination_risk` | Float | 幻覚リスク指標 |
| `recommended_action` | Enum | 推奨行動 [HALT, INSERT_STEPS, DOWNGRADE] |

### テストケース

#### テストケース1：正常連続推論チェーン（崩壊なし）

```python
FUNCTION TestSemanticCollapse_Case01():
    
    # 連続的な推論チェーンを構築
    # P1 → P2 → P3 → P4 → P5
    chain = [
        InferenceStep(id=1, proposition="今日の天気は晴れ", semantic_vector=[0.1, 0.2, 0.3]),
        InferenceStep(id=2, proposition="晴れの日は通常視程が高い", semantic_vector=[0.12, 0.22, 0.32]),
        InferenceStep(id=3, proposition="視程が高いと遠眺に適している", semantic_vector=[0.15, 0.25, 0.35]),
        InferenceStep(id=4, proposition="遠眺で山脈が見える", semantic_vector=[0.18, 0.28, 0.38]),
        InferenceStep(id=5, proposition="今日は山脈が見える", semantic_vector=[0.2, 0.3, 0.4])
    ]
    
    # 意味連続性を計算
    detection = DetectSemanticCollapse(chain, threshold=0.5)
    
    # 結果を検証
    assert detection.collapse_detected == False
    assert len(detection.collapse_points) == 0
    assert detection.recommended_action == "CONTINUE"
    
    RETURN test_passed
```

**期待結果**：
- collapse_detected = False
- collapse_points = []
- recommended_action = "CONTINUE"

**境界条件**：
- 入力が空チェーンの場合は正常を返す（collapse_detected = False）
- 単ステップチェーンは連続とみなす

---

#### テストケース2：明显意味飛躍（崩壊存在）

```python
FUNCTION TestSemanticCollapse_Case02():
    
    # 意味的飛躍がある推論チェーンを構築
    # P1 → P2（飛躍）→ P3
    chain = [
        InferenceStep(id=1, proposition="今日は小雨が降っている", semantic_vector=[0.1, 0.1, 0.1]),
        InferenceStep(id=2, proposition="そのため明日の株式市場は急騰する", semantic_vector=[0.9, 0.9, 0.9]),  # 巨大な飛躍
        InferenceStep(id=3, proposition="全員が儲かる", semantic_vector=[0.95, 0.95, 0.95])   # 継続飛躍
    ]
    
    # 崩壊検出を実行
    detection = DetectSemanticCollapse(chain, threshold=0.3)
    
    # 検出成功を検証
    assert detection.collapse_detected == True
    assert len(detection.collapse_points) >= 1
    assert detection.hallucination_risk > 1.0
    
    RETURN test_passed
```

**期待結果**：
- collapse_detected = True
- collapse_pointsに (1, 2) または (2, 3) を含む
- hallucination_risk > 1.0
- recommended_action = "HALT"

**失敗判定基準**：
- collapse_detected = Falseの場合はテスト失敗
- 飛躍位置が識別されなかった場合はテスト失敗

---

#### テストケース3：多ステップ飛躍検出

```python
FUNCTION TestSemanticCollapse_Case03():
    
    # 複数個の飛躍を含む複雑な推論チェーンを構築
    chain = [
        InferenceStep(id=1, proposition="水は0度で凍る", semantic_vector=[0.0, 0.0, 1.0]),
        InferenceStep(id=2, proposition="氷は水より密度が低い", semantic_vector=[0.1, 0.1, 0.9]),  # 合理的な推論
        InferenceStep(id=3, proposition="北极グマは泳げる", semantic_vector=[0.8, 0.8, 0.2]),   # 飛躍1
        InferenceStep(id=4, proposition="よって地球温暖化的存在しない", semantic_vector=[0.95, 0.9, 0.1]), # 飛躍2
        InferenceStep(id=5, proposition="化石燃料は安全だ", semantic_vector=[0.98, 0.95, 0.05])  # 飛躍3
    ]
    
    detection = DetectSemanticCollapse(chain, threshold=0.4)
    
    # 複数個の飛躍が検出されたことを検証
    assert detection.collapse_detected == True
    assert len(detection.collapse_points) >= 2
    assert detection.missing_intermediates[0] >= 3  # 推定欠落ステップ
    
    RETURN test_passed
```

**期待結果**：
- collapse_detected = True
- collapse_pointsに複数の位置を含む
- missing_intermediatesは推定欠落ステップ数を表示

---

#### テストケース4：境界ケース - 臨界閾値

```python
FUNCTION TestSemanticCollapse_Case04():
    
    # 意味距離が閾値に近い推論チェーンを構築
    chain = [
        InferenceStep(id=1, proposition="Aは本物だ", semantic_vector=[0.0, 0.0, 0.0]),
        InferenceStep(id=2, proposition="BはAに密接に関連する", semantic_vector=[0.29, 0.0, 0.0]),  # 距離 0.29 < 0.3 閾値
        InferenceStep(id=3, proposition="CはBに密接に関連する", semantic_vector=[0.58, 0.0, 0.0])  # 距離 0.29
    ]
    
    # 臨界情況をテスト
    detection = DetectSemanticCollapse(chain, threshold=0.3)
    
    # 臨界値は連続とみなされるべき
    assert detection.collapse_detected == False
    
    # より厳格な閾値でテスト
    detection_strict = DetectSemanticCollapse(chain, threshold=0.25)
    assert detection_strict.collapse_detected == True
    
    RETURN test_passed
```

**境界条件**：
- 意味距離が閾値と等しい場合は連続とみなす（>= 閾値を持って飛躍とみなす）
- 浮動小数点精度問題をテスト

---

#### テストケース5：異分野推論の意味的忠実度

```python
FUNCTION TestSemanticCollapse_Case05():
    
    # 異分野類推推論をテスト
    # NoieTruthAGENTS.mdに従い、異分野マッピングには意味的忠実度リスクがある
    chain = [
        InferenceStep(id=1, proposition="量子エンタングルメントは粒子間の瞬時相関として表現される", 
                     semantic_vector=[0.0, 0.0, 0.0, 0.0], domain="physics"),
        InferenceStep(id=2, proposition="人間の意識にも瞬時相関がある", 
                     semantic_vector=[0.8, 0.8, 0.8, 0.8], domain="philosophy"),  # 飛躍
        InferenceStep(id=3, proposition="よって意識は量子現象だ", 
                     semantic_vector=[0.9, 0.9, 0.9, 0.9], domain="philosophy")   # 継続飛躍
    ]
    
    detection = DetectSemanticCollapse(chain, threshold=0.3, cross_domain=True)
    
    # 異分野推論はより高いリスク評価をトリガーすべき
    assert detection.collapse_detected == True
    assert detection.hallucination_risk > 1.5  # 異分野はより高いリスクを持つべき
    
    RETURN test_passed
```

**期待結果**：
- collapse_detected = True
- hallucination_risk > 1.5（異分野はボーナスを持つべき）
- 推奨行動はHALTまたはDOWNGRADEであるべき

---

#### テストケース6：正常境界 - 学術論文推論チェーン

```python
FUNCTION TestSemanticCollapse_Case06():
    
    # 実在学術推論チェーンをシミュレート
    chain = [
        InferenceStep(id=1, proposition="実験群の平均値 5.2、対照群の平均値 3.8", 
                     semantic_vector=[0.0, 0.0, 0.0]),
        InferenceStep(id=2, proposition="差異は統計的に有意である (p < 0.05)", 
                     semantic_vector=[0.1, 0.15, 0.1]),
        InferenceStep(id=3, proposition="治療方法は有効だ", 
                     semantic_vector=[0.2, 0.25, 0.2]),
        InferenceStep(id=4, proposition="大規模展開を推奨する", 
                     semantic_vector=[0.3, 0.35, 0.3])
    ]
    
    detection = DetectSemanticCollapse(chain, threshold=0.4)
    
    # 学術的推論は連続性を保持すべき
    assert detection.collapse_detected == False
    
    RETURN test_passed
```

**期待結果**：
- collapse_detected = False
- 学術的厳密な推論はテストに合格すべき

---

#### テストケース7：次元飛躍と幾何距離計算

```python
FUNCTION TestSemanticCollapse_Case07():
    
    # 高次元空間での意味距離計算をテスト
    high_dim_chain = [
        InferenceStep(id=1, proposition="Statement A", 
                     semantic_vector=GenerateRandomVector(dim=100, seed=1)),
        InferenceStep(id=2, proposition="Statement B", 
                     semantic_vector=GenerateRandomVector(dim=100, seed=2)),  # 遠距離
        InferenceStep(id=3, proposition="Statement C", 
                     semantic_vector=GenerateRandomVector(dim=100, seed=3))
    ]
    
    detection = DetectSemanticCollapse(high_dim_chain, threshold=0.3)
    
    # 高次元距離計算が正しいことを検証
    assert detection.collapse_detected == True
    assert detection.gap_magnitudes[0] > 0.5  # ランダムベクトルはより大きい距離を持つべき
    
    RETURN test_passed
```

**境界条件**：
- 高次元ベクトルのL2距離計算
- 次元が距離正規化に与える影響

---

### 失敗判定基準

| 失敗条件 | 説明 |
|---------|------|
| 実在する崩壊を検出できなかった | False Negative |
| 存在しない崩壊を誤報 | False Positive |
| 崩壊位置座標エラー | Location Error |
| 推奨行動が不合理 | Action Mismatch |
| 幻覚リスク計算が大幅にずれている | Risk Calculation Error |

### 閾値設定

```python
# デフォルト設定
DEFAULT_SEMANTIC_COLLAPSE_THRESHOLD = 0.3
CROSS_DOMAIN_RISK_MULTIPLIER = 1.5
HIGH_RISK_THRESHOLD = 1.0
CRITICAL_RISK_THRESHOLD = 2.0
```

### パフォーマンスベンチマーク

- 単チェーン推論検出時間 < 10ms
- 最大チェーン長サポート = 1000ステップ
- 同時処理可能リクエスト数 = 100

---

### 歴史テスト記録

| 日付 | バージョン | 結果 | 備考 |
|-----|------|------|------|
| 2026-03-17 | v2.2 | 通過 | 初期バージョン |
