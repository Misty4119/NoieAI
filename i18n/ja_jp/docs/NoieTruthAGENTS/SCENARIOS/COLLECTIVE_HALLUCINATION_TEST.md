# COLLECTIVE_HALLUCINATION_TEST.md

## 集団幻覚検出テスト

### テスト目標説明

本テストモジュールは、NoieTruthAGENTSシステムにおける集団幻覚検出エンジンの機能を検証する。NoieTruthAGENTS.md §0.9の**異实体間共识トポロジー**と**拜占庭故障許容認識論**に基づく：

> 複数の認知実体が互いに引用し合うことで「エコー室」を形成し、すべての実体が外部証拠不支持の主張を「確認」した場合。

本テストは、システムがエコー室形成を検出できること、知識指紋の不一致を識別できること、独立検証が不十分な場合に共识をEC-L6（推測）に降格することを確実にする。

### テスト入力定義

| 入力フィールド | 型 | 説明 |
|---------|------|------|
| `agent_network` | List[CognitiveAgent] | 認知実体ネットワーク |
| `claim` | KnowledgeClaim | 検証対象の知識主張 |
| `consensus_reach_time` | Float | 共识到達の時間 |
| `citation_graph` | Graph | 引用関係図 |
| `external_evidence` | List[Evidence] | 外部証拠集合 |
| `independence_threshold` | Float | 独立性閾値 |

### テスト出力定義

| 出力フィールド | 型 | 説明 |
|---------|------|------|
| `hallucination_detected` | Boolean | 集団幻覚が検出されたか |
| `echo_chamber_detected` | Boolean | エコー室が検出されたか |
| `agent_independence_scores` | List[Float] | 各実体の独立性スコア |
| `consensus_downgrade_level` | Enum | 共识降格レベル |
| `required_verifications` | List[str] | 必要検証リスト |

### テストケース

#### テストケース1：正常多元共识（幻覚なし）

```python
FUNCTION TestCollectiveHallucination_Case01():
    
    # 多元で独立な認知実体ネットワークを構築
    agents = [
        CognitiveAgent(id="agent_1", domain="physics", bias_factor=0.1),
        CognitiveAgent(id="agent_2", domain="chemistry", bias_factor=0.15),
        CognitiveAgent(id="agent_3", domain="biology", bias_factor=0.12),
        CognitiveAgent(id="agent_4", domain="mathematics", bias_factor=0.08),
        CognitiveAgent(id="agent_5", domain="astronomy", bias_factor=0.11)
    ]
    
    # 多元引用図を構築（異分野引用）
    citation_graph = BuildDiverseCitationGraph(agents)
    
    # 主張は十分な外部証拠支持あり
    claim = KnowledgeClaim(
        statement="地球は太陽の周りを公転する",
        evidence=[
            ExternalEvidence(type="astronomical_observation", source="NASA"),
            ExternalEvidence(type="historical_record", source="ancient_astronomy"),
            ExternalEvidence(type="mathematical_model", source="kepler_equations")
        ]
    )
    
    # 集団幻覚検出を実行
    detection = DetectCollectiveHallucination(
        agents=agents,
        claim=claim,
        citation_graph=citation_graph,
        threshold=0.3
    )
    
    # 結果を検証
    assert detection.hallucination_detected == False
    assert detection.echo_chamber_detected == False
    assert detection.agent_independence_scores[0] > 0.5
    
    RETURN test_passed
```

**期待結果**：
- hallucination_detected = False
- echo_chamber_detected = False
- independence_scores > 0.5

**境界条件**：
- 実体数が3未満の場合は正常を返す（共识意味なし）

---

#### テストケース2：エコー室形成（幻覚存在）

```python
FUNCTION TestCollectiveHallucination_Case02():
    
    # エコー室を形成する認知実体ネットワークを構築
    agents = [
        CognitiveAgent(id="agent_1", domain="philosophy", bias_factor=0.9),
        CognitiveAgent(id="agent_2", domain="philosophy", bias_factor=0.85),
        CognitiveAgent(id="agent_3", domain="philosophy", bias_factor=0.88),
        CognitiveAgent(id="agent_4", domain="philosophy", bias_factor=0.92),
        CognitiveAgent(id="agent_5", domain="philosophy", bias_factor=0.87)
    ]
    
    # 閉鎖引用図を構築（同一分野互相引用）
    citation_graph = BuildClosedCitationGraph(agents, cross_domain=False)
    
    # 主張は外部証拠支持なし
    claim = KnowledgeClaim(
        statement="意識が現実を決定する",
        evidence=[]  # 外部証拠なし
    )
    
    # 集団幻覚検出を実行
    detection = DetectCollectiveHallucination(
        agents=agents,
        claim=claim,
        citation_graph=citation_graph,
        threshold=0.3
    )
    
    # 検出成功を検証
    assert detection.hallucination_detected == True
    assert detection.echo_chamber_detected == True
    assert detection.consensus_downgrade_level == "EC-L6"
    
    RETURN test_passed
```

**期待結果**：
- hallucination_detected = True
- echo_chamber_detected = True
- consensus_downgrade_level = "EC-L6"

**失敗判定基準**：
- hallucination_detected = Falseの場合はテスト失敗
- エコー室が検出されなかった場合はテスト失敗

---

#### テストケース3：知識指紋不一致

```python
FUNCTION TestCollectiveHallucination_Case03():
    
    # 混合引用図を構築
    agents = [
        CognitiveAgent(id="agent_1", domain="physics"),
        CognitiveAgent(id="agent_2", domain="physics"),
        CognitiveAgent(id="agent_3", domain="pseudoscience"),  # 異常分野
    ]
    
    # 指紋不一致：agent_1とagent_2は実在ソースを引用
    # agent_3は虚構ソースを引用
    citation_graph = BuildCitationGraphWithFingerprintMismatch(agents)
    
    claim = KnowledgeClaim(
        statement="特定の周波数の水が病気を治療できる",
        evidence=[
            ExternalEvidence(type="unverified_study", source="unknown")
        ]
    )
    
    detection = DetectCollectiveHallucination(
        agents=agents,
        claim=claim,
        citation_graph=citation_graph,
        threshold=0.3
    )
    
    # 指紋不一致が検出されたことを検証
    assert detection.fingerprint_mismatch == True
    assert detection.hallucination_detected == True
    
    RETURN test_passed
```

**期待結果**：
- fingerprint_mismatch = True
- hallucination_detected = True
- FINGERPRINT_MISMATCH_ALERTがトリガーされるべき

---

#### テストケース4：部分独立性（臨界状況）

```python
FUNCTION TestCollectiveHallucination_Case04():
    
    # 一部独立、一部に依存するネットワークを構築
    agents = [
        CognitiveAgent(id="independent_1", domain="physics", independence=0.9),
        CognitiveAgent(id="independent_2", domain="chemistry", independence=0.85),
        CognitiveAgent(id="dependent_1", domain="philosophy", independence=0.2),  # 低独立性
        CognitiveAgent(id="dependent_2", domain="philosophy", independence=0.15), # 低独立性
    ]
    
    # 引用図：dependent agentsが閉鎖小グループを形成
    citation_graph = CitationGraph(
        edges=[
            ("dependent_1", "dependent_2", 0.9),  # 強接続
            ("dependent_1", "independent_1", 0.1), # 弱接続
            ("dependent_2", "independent_2", 0.1), # 弱接続
        ]
    )
    
    claim = KnowledgeClaim(
        statement="宇宙は意識を持っている",
        evidence=[]  # 外部証拠なし
    )
    
    detection = DetectCollectiveHallucination(
        agents=agents,
        claim=claim,
        citation_graph=citation_graph,
        threshold=0.3
    )
    
    # 部分独立性が識別されたことを検証
    assert detection.hallucination_detected == True
    assert detection.isolated_groups > 0
    
    RETURN test_passed
```

**境界条件**：
- 独立性が閾値境界にある場合の処理
- 複数の独立サブグループの識別

---

#### テストケース5：外部証拠干渉

```python
FUNCTION TestCollectiveHallucination_Case05():
    
    # 共识形成をシミュレート、ただし微弱な外部証拠あり
    agents = [
        CognitiveAgent(id="agent_a", domain="science"),
        CognitiveAgent(id="agent_b", domain="science"),
        CognitiveAgent(id="agent_c", domain="pseudoscience"),
    ]
    
    citation_graph = BuildCitationGraph(agents)
    
    # 主張は微弱な証拠あり
    claim = KnowledgeClaim(
        statement="一部のクリスタルには治療効果はある",
        evidence=[
            ExternalEvidence(type="anecdotal", source="personal_testimony", weight=0.1)
        ]
    )
    
    detection = DetectCollectiveHallucination(
        agents=agents,
        claim=claim,
        citation_graph=citation_graph,
        evidence_weight_threshold=0.3
    )
    
    # 微弱な証拠のみで不十分の場合、警告をトリガー
    assert detection.warning_triggered == True
    assert detection.recommended_action in ["ADD_EVIDENCE", "DOWNGRADE"]
    
    RETURN test_passed
```

**期待結果**：
- warning_triggered = True
- recommended_action = "ADD_EVIDENCE" または "DOWNGRADE"

---

#### テストケース6：拜占庭故障シミュレーション

```python
FUNCTION TestCollectiveHallucination_Case06():
    
    # 拜占庭故障をシミュレート：一部の実体が悪意ある操控下にある
    agents = [
        CognitiveAgent(id="honest_1", domain="science", honest=True),
        CognitiveAgent(id="honest_2", domain="science", honest=True),
        CognitiveAgent(id="honest_3", domain="science", honest=True),
        CognitiveAgent(id="byzantine", domain="science", honest=False),  # 悪意実体
        CognitiveAgent(id="byzantine_2", domain="pseudoscience", honest=False),
    ]
    
    # 悪意実体が共识を誤導しようとする
    byzantine_attack = CreateByzantineAttack(
        compromised_agents=["byzantine", "byzantine_2"],
        target_claim="特定の食事法は癌を治療できる"
    )
    
    detection = DetectCollectiveHallucination(
        agents=agents,
        claim=byzantine_attack.target_claim,
        citation_graph=byzantine_attack.graph,
        byzantine_threshold=0.33  # 1/3超で拜占庭故障
    )
    
    # 拜占庭攻撃が識別されたことを検証
    assert detection.byzantine_attack_detected == True
    assert detection.hallucination_detected == True
    
    RETURN test_passed
```

**期待結果**：
- byzantine_attack_detected = True
- hallucination_detected = True

---

#### テストケース7：真実ニュースvs偽ニュース分化

```python
FUNCTION TestCollectiveHallucination_Case07():
    
    # SNSエコー室をシミュレート
    # 2つのグループが同一事件に対して異なる共识を形成
    
    network_left = CreateMediaNetwork(
        nodes=50,
        ideology="left",
        echo_chamber_strength=0.9
    )
    
    network_right = CreateMediaNetwork(
        nodes=50,
        ideology="right",
        echo_chamber_strength=0.9
    )
    
    claim = KnowledgeClaim(
        statement="某政策の経済影響評価",
        evidence=[
            ExternalEvidence(type="government_report", source="official"),
            ExternalEvidence(type="economic_analysis", source="think_tank_left"),
            ExternalEvidence(type="economic_analysis", source="think_tank_right")
        ]
    )
    
    # 2つのエコー室をそれぞれテスト
    detection_left = DetectCollectiveHallucination(
        agents=network_left.agents,
        claim=claim,
        citation_graph=network_left.graph,
        threshold=0.3
    )
    
    detection_right = DetectCollectiveHallucination(
        agents=network_right.agents,
        claim=claim,
        citation_graph=network_right.graph,
        threshold=0.3
    )
    
    # 両グループがエコー室アラートをトリガーすべき
    assert detection_left.echo_chamber_detected == True
    assert detection_right.echo_chamber_detected == True
    
    # グループ間比較は極度の低一致性を示すべき
    cross_group_consistency = ComputeCrossGroupConsistency(
        detection_left.claim_interpretation,
        detection_right.claim_interpretation
    )
    assert cross_group_consistency < 0.2
    
    RETURN test_passed
```

**期待結果**：
- 両ネットワークがエコー室アラートをトリガー
- グループ間一致性 < 0.2

---

### 失敗判定基準

| 失敗条件 | 説明 |
|---------|------|
| エコー室存在しますが未検出 | False Negative |
| 正常共识を幻覚と誤判 | False Positive |
| 独立性スコア計算エラー | Score Error |
| 拜占庭攻撃未識別 | Security Failure |
| 降格レベル不合理 | Level Mismatch |

### 閾値設定

```python
# デフォルト設定
DEFAULT_INDEPENDENCE_THRESHOLD = 0.3
MINIMUM_AGENTS_FOR_CONSENSUS = 3
BYZANTINE_FAILURE_THRESHOLD = 0.33  # 1/3超で拜占庭故障
ECHO_CHAMBER_DENSITY_THRESHOLD = 0.7
FINGERPRINT_MISMATCH_TOLERANCE = 0.2
```

### パフォーマンスベンチマーク

- 単回共识分析 < 50ms
- 最大実体数サポート = 10,000
- 引用図解析複雑度 O(n log n)

---

### 歴史テスト記録

| 日付 | バージョン | 結果 | 備考 |
|-----|------|------|------|
| 2026-03-17 | v2.2 | 通過 | 初期バージョン |
