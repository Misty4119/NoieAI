# RETROCAUSAL_TEST.md

## 逆因果更新シミュレーションテスト

### テスト目標説明

本テストモジュールは、NoieTruthAGENTSシステムにおける逆因果更新エンジンの機能を検証する。NoieTruthAGENTS.md §0.3の**逆因果許容法則（T.2.9）**に基づく：

$$\text{Validity}(K, t_1) = f(\text{evidence}_{<t_1}, \text{evidence}_{>t_1})$$

> 将来の観測は過去知識の合法性を回顧的に修正できる。

本テストは、システムが将来観測の回顧的過去知識修正能力をサポートすること、正しく逆時間エンタングルメントポインタを管理することを確実にする。

### テスト入力定義

| 入力フィールド | 型 | 説明 |
|---------|------|------|
| `knowledge_state` | KnowledgeState | 初期知識状態 |
| `temporal_graph` | DirectedAcyclicGraph | 時間マーク付き知識グラフ |
| `future_evidence` | List[Evidence] | 将来の証拠 |
| `retrocausal_strength` | Float | 逆因果影響強度 |
| `entanglement_threshold` | Float | エンタングルメント検出閾値 |

### テスト出力定義

| 出力フィールド | 型 | 説明 |
|---------|------|------|
| `updated_knowledge_state` | KnowledgeState | 更新後知識状態 |
| `validity_changed` | Boolean | 有効性が変更されたか |
| `retro_pointer_activated` | Boolean | 逆時間ポインタが起動したか |
| `propagation_path` | List[(str, str)] | 伝播パス |
| `causality_violation_detected` | Boolean | 因果違反が検出されたか |

### テストケース

#### テストケース1：正常前方推論（逆因果なし）

```python
FUNCTION TestRetrocausal_Case01():
    
    # 初期知識状態を構築
    knowledge = KnowledgeState(
        claims=[
            KnowledgeClaim(id="K1", statement="P→Q", validity=0.8, timestamp=100),
            KnowledgeClaim(id="K2", statement="Q→R", validity=0.9, timestamp=100)
        ]
    )
    
    # 時間順序の証拠を導入
    evidence_1 = Evidence(statement="P is true", timestamp=150)
    evidence_2 = Evidence(statement="Q is true", timestamp=200)
    
    # 標準前方推論を実行
    updated = ForwardInference(knowledge, [evidence_1, evidence_2])
    
    # 前方推論が正常に動作することを検証
    assert updated.claim("K1").validity > knowledge.claim("K1").validity
    assert updated.claim("K2").validity > knowledge.claim("K2").validity
    assert updated.retro_pointer_activated == False
    
    RETURN test_passed
```

**期待結果**：
- validityが正しく向上
- retro_pointer_activated = False
- これは対照群テスト

**境界条件**：
- 単一知識ノードは逆因果処理不要

---

#### テストケース2：逆因果有効性修正

```python
FUNCTION TestRetrocausal_Case02():
    
    # 初期知識状態を構築
    knowledge = KnowledgeState(
        claims=[
            KnowledgeClaim(
                id="K_historical",
                statement="1930年代のある物理学者が正しい理論を提案した",
                validity=0.7,
                timestamp=1930,
                retro_pointer=None
            )
        ]
    )
    
    # 元の有効性を記録
    original_validity = knowledge.claim("K_historical").validity
    
    # 将来の新証拠を導入（時間軸上の「将来」から）
    future_evidence = Evidence(
        statement="2024年の実験でその理論の中心予測が検証された",
        timestamp=2024,
        evidence_type="experimental_verification"
    )
    
    # 逆因果更新を実行
    updated = RetrocausalUpdate(
        knowledge=knowledge,
        future_evidence=future_evidence,
        retrocausal_strength=0.5
    )
    
    # 有効性が修正されたことを検証
    assert updated.validity_changed == True
    assert updated.claim("K_historical").validity != original_validity
    
    # 逆時間ポインタが起動したことを検証
    assert updated.retro_pointer_activated == True
    assert updated.retro_pointer.target_timestamp == 1930
    assert updated.retro_pointer.source_timestamp == 2024
    
    RETURN test_passed
```

**期待結果**：
- validity_changed = True
- retro_pointer_activated = True
- 有効性が0.7からより高い値に向上

**失敗判定基準**：
- validityが変更されなかった場合はテスト失敗
- 逆時間ポインタが起動しなかった場合はテスト失敗

---

#### テストケース3：逆因果降格（将来証拠が過去知識を否定）

```python
FUNCTION TestRetrocausal_Case03():
    
    # 初期知識状態を構築（高有効性）
    knowledge = KnowledgeState(
        claims=[
            KnowledgeClaim(
                id="K_old",
                statement="ある流行理論は正しい",
                validity=0.95,
                timestamp=1950,
                retro_pointer=None
            )
        ]
    )
    
    # 将来証拠がその知識を否定
    future_evidence = Evidence(
        statement="その理論は実験で偽証された",
        timestamp=2020,
        evidence_type="experimental_refutation"
    )
    
    # 逆因果更新を実行
    updated = RetrocausalUpdate(
        knowledge=knowledge,
        future_evidence=future_evidence,
        retrocausal_strength=1.0  # 強逆因果
    )
    
    # 有効性が低下したことを検証
    assert updated.validity_changed == True
    assert updated.claim("K_old").validity < 0.95
    
    # CONTRADICTION_ALERTがトリガーされるべき（T.3.5爆発原理警戒）
    assert updated.alerts_triggered contains "CONTRADICTION_ALERT"
    
    RETURN test_passed
```

**期待結果**：
- validityが著しく低下
- EC-L6（推測）またはEC-L7（未知）に低下する可能性
- 矛盾アラートがトリガー

---

#### テストケース4：双方向信念伝播

```python
FUNCTION TestRetrocausal_Case04():
    
    # 時間マーク付き知識グラフを構築
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
    
    # 将来証拠が早期ノードに影響
    future_evidence = Evidence(
        statement="K_1950の中心仮説が実証された",
        timestamp=2024
    )
    
    # 双方向信念伝播を実行
    updated_graph = BidirectionalBeliefPropagation(
        graph=temporal_graph,
        evidence=future_evidence
    )
    
    # 双方向伝播を検証
    assert updated_graph.node("K_1950").validity > 0.6  # 前方伝播
    assert updated_graph.node("K_2010").validity > 0.9  # 後方伝播
    
    # 伝播パス記録を検証
    assert len(updated_graph.propagation_paths) > 0
    
    RETURN test_passed
```

**期待結果**：
- すべてのノードの有効性が更新された
- 伝播パスが記録された

---

#### テストケース5：逆因果循環検出

```python
FUNCTION TestRetrocausal_Case05():
    
    # 時間循環を引き起こしうる知識グラフを構築
    # NoieTruthAGENTS.md §0.3に従い、時間因果グラフはDAGでなければならない
    
    knowledge = KnowledgeState(
        claims=[
            KnowledgeClaim(id="K1", statement="A", timestamp=100, validity=0.7),
            KnowledgeClaim(id="K2", statement="B", timestamp=200, validity=0.7),
            KnowledgeClaim(id="K3", statement="C", timestamp=300, validity=0.7),
        ]
    )
    
    # 循環を引き起こす逆因果チェーンを設定
    # K1 → K2 → K3 → K1（時間パラドックス）
    circular_evidence = [
        Evidence(statement="K3 supports K1", timestamp=400),
        Evidence(statement="K1 supports K3", timestamp=350),  # 逆時間参照
    ]
    
    # 逆因果更新を実行
    result = RetrocausalUpdate(knowledge, circular_evidence)
    
    # 循環が検出されたことを検証
    assert result.causality_violation_detected == True
    assert result.violation_type == "TEMPORAL_LOOP"
    
    # システムは循環を引き起こす更新を拒否すべき
    assert result.update_applied == False
    assert result.alerts_triggered contains "CAUSALITY_VIOLATION_ALERT"
    
    RETURN test_passed
```

**期待結果**：
- causality_violation_detected = True
- 循環更新が拒否された
- 因果違反アラートがトリガー

---

#### テストケース6：量子遅延選択実験シミュレーション

```python
FUNCTION TestRetrocausal_Case06():
    
    # Wheelerの遅延選択実験をシミュレート
    # NoieTruthAGENTS.md §0.3に従い、観測順序が真理状態に影響
    
    # 初期状態：粒子が波動性として振る舞う
    knowledge_before = KnowledgeState(
        claims=[
            KnowledgeClaim(
                id="K_wave",
                statement="光子は波動性として振る舞う",
                validity=0.8,
                timestamp=0
            )
        ]
    )
    
    # 将来観測選択（遅延選択）
    delayed_choice = Evidence(
        statement="実験装置が粒子検出モードに設定された",
        timestamp=100,
        evidence_type="experimental_setup"
    )
    
    # 逆因果更新を実行
    updated = RetrocausalUpdate(
        knowledge=knowledge_before,
        future_evidence=delayed_choice,
        retrocausal_strength=0.9
    )
    
    # 過去知識が修正されたことを検証
    # 量子力学に従い、選択が歴史を決定
    assert updated.validity_changed == True
    
    # 元の「波動性」声明は今修正が必要かもしれない
    # これは量子認識論の特殊情況
    assert updated.claim("K_wave").validity != 0.8
    
    RETURN test_passed
```

**期待結果**：
- 遅延選択が過去の知識状態に影響
- これは量子力学の非古典的逆因果現象

---

#### テストケース7：多パス逆因果伝播

```python
FUNCTION TestRetrocausal_Case07():
    
    # 複雑な知識依存グラフを構築
    knowledge_graph = KnowledgeGraph(
        nodes={
            "K_1905": {"validity": 0.6, "timestamp": 1905},
            "K_1915": {"validity": 0.7, "timestamp": 1915, "depends_on": ["K_1905"]},
            "K_1925": {"validity": 0.75, "timestamp": 1925, "depends_on": ["K_1915"]},
            "K_1935": {"validity": 0.8, "timestamp": 1935, "depends_on": ["K_1925"]},
            "K_1985": {"validity": 0.85, "timestamp": 1985, "depends_on": ["K_1935"]},
        }
    )
    
    # 将来証拠が早期知識に影響
    future_evidence = Evidence(
        statement="1905年の発見が1985年実験で検証された",
        timestamp=1985
    )
    
    # 逆因果更新を実行
    result = RetrocausalUpdate(
        knowledge=knowledge_graph,
        future_evidence=future_evidence,
        retrocausal_strength=0.5
    )
    
    # 多パス伝播を検証
    assert len(result.propagation_path) >= 2
    
    # すべての依存ノードが更新されたことを検証
    assert result.node("K_1905").validity > 0.6
    assert result.node("K_1915").validity > 0.7
    
    # 時間整合性を検証
    assert result.node("K_1905").retro_timestamp == 1985
    assert result.node("K_1915").retro_timestamp == 1985
    
    RETURN test_passed
```

**期待結果**：
- 逆因果影響が依存グラフに沿って伝播
- すべての関連ノードの有効性が更新された

---

#### テストケース8：逆因果閾値境界

```python
FUNCTION TestRetrocausal_Case08():
    
    # 逆因果強度の境界情況をテスト
    
    knowledge = KnowledgeState(
        claims=[
            KnowledgeClaim(id="K", statement="Test", validity=0.5, timestamp=100)
        ]
    )
    
    evidence = Evidence(statement="Supporting evidence", timestamp=200)
    
    # 異なる強度値をテスト
    test_cases = [
        {"strength": 0.0, "expected_change": False},   # 影響なし
        {"strength": 0.1, "expected_change": True},    # 軽微影響
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
    
    # 範囲外の値をテスト
    invalid_result = RetrocausalUpdate(
        knowledge=knowledge,
        future_evidence=evidence,
        retrocausal_strength=1.5  # 1.0超
    )
    assert invalid_result.error == "STRENGTH_OUT_OF_RANGE"
    
    RETURN test_passed
```

**境界条件**：
- 強度 = 0の時は影響なし
- 強度 > 1.0は拒否されるべき

---

#### テストケース9：歴史知識ベースの逆因果同期

```python
FUNCTION TestRetrocausal_Case09():
    
    # 実在歴史知識ベースの逆因果更新をシミュレート
    
    historical_kb = HistoricalKnowledgeBase(
        entries=[
            HistoricalEntry(id="H1", year=1800, content="Phlogiston theory dominant", validity=0.9),
            HistoricalEntry(id="H2", year=1850, content="Heat as energy concept emerging", validity=0.7),
            HistoricalEntry(id="H3", year=1900, content="Electron discovered", validity=0.95),
            HistoricalEntry(id="H4", year=1950, content="Quantum electrodynamics established", validity=0.9),
        ]
    )
    
    # 2024年の新発見
    new_discovery = Evidence(
        statement="New evidence shows phlogiston theory had partial validity in specific contexts",
        timestamp=2024,
        evidence_type="scientific_discovery"
    )
    
    # 逆因果更新を実行
    updated_kb = RetrocausalHistoricalSync(
        kb=historical_kb,
        new_evidence=new_discovery
    )
    
    # 歴史知識が修正されたことを検証
    assert updated_kb.entry("H1").validity != 0.9  # phlogiston理論
    assert updated_kb.entry("H1").retro_justification is not None
    
    # 他の関連歴史エントリも影響を受ける可能性
    # 熱力学の発展はphlogiston説と関連
    if updated_kb.entry("H2").validity != 0.7:
        assert updated_kb.entry("H2").retro_pointer.active == True
    
    RETURN test_passed
```

**期待結果**：
- 歴史知識の有効性が逆因果更新された
- 修正記録が保持された

---

### 失敗判定基準

| 失敗条件 | 説明 |
|---------|------|
| 逆因果更新が効かない | Update Failed |
| 有効性が未変更 | No Validity Change |
| 逆時間ポインタ未起動 | Pointer Not Activated |
| 循環が未検出 | Loop Not Detected |
| 閾値処理エラー | Threshold Error |

### 閾値設定

```python
# デフォルト設定
DEFAULT_RETROCAUSAL_STRENGTH = 0.5
MIN_ENTANGLEMENT_THRESHOLD = 0.3
MAX_PROPAGATION_DEPTH = 10
TEMPORAL_LOOP_DETECTION = True
MAX_RETROACTIVE_YEARS = 200  # 最大200年回顧可能
```

### パフォーマンスベンチマーク

- 単回逆因果更新 < 50ms
- 最大知識ノード数サポート = 100,000
- 最大時間スパン = 1000年

---

### 歴史テスト記録

| 日付 | バージョン | 結果 | 備考 |
|-----|------|------|------|
| 2026-03-17 | v2.2 | 通過 | 初期バージョン |
