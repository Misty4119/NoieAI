# COLLECTIVE_HALLUCINATION_TEST.md

## 集體幻覺偵測測試

### 測試目標說明

本測試模組驗證 NoieTruthAGENTS 系統中集體幻覺偵測引擎的功能。根據 NoieTruthAGENTS.md §0.9 中的**跨實體共識拓撲**與**拜占庭容錯知識論**：

> 當多個認知實體因互相引用而形成「回音室」，所有實體都「確認」了一個實際上無外部證據支持的宣稱。

此測試確保系統能偵測回音室形成、識別知識指紋不匹配，並在獨立驗證不足時將共識降級為 EC-L6（推測）。

### 測試輸入定義

| 輸入欄位 | 類型 | 描述 |
|---------|------|------|
| `agent_network` | List[CognitiveAgent] | 認知實體網路 |
| `claim` | KnowledgeClaim | 待驗證的知識宣稱 |
| `consensus_reach_time` | Float | 達到共識的時間 |
| `citation_graph` | Graph | 引用關係圖 |
| `external_evidence` | List[Evidence] | 外部證據集合 |
| `independence_threshold` | Float | 獨立性閾值 |

### 測試輸出定義

| 輸出欄位 | 類型 | 描述 |
|---------|------|------|
| `hallucination_detected` | Boolean | 是否偵測到集體幻覺 |
| `echo_chamber_detected` | Boolean | 是否偵測到回音室 |
| `agent_independence_scores` | List[Float] | 各實體獨立性分數 |
| `consensus_downgrade_level` | Enum | 共識降級層級 |
| `required_verifications` | List[str] | 所需驗證清單 |

### 測試案例

#### 測試案例 1：正常多元共識（無幻覺）

```python
FUNCTION TestCollectiveHallucination_Case01():
    
    # 建立多元且獨立的認知實體網路
    agents = [
        CognitiveAgent(id="agent_1", domain="physics", bias_factor=0.1),
        CognitiveAgent(id="agent_2", domain="chemistry", bias_factor=0.15),
        CognitiveAgent(id="agent_3", domain="biology", bias_factor=0.12),
        CognitiveAgent(id="agent_4", domain="mathematics", bias_factor=0.08),
        CognitiveAgent(id="agent_5", domain="astronomy", bias_factor=0.11)
    ]
    
    # 建立多元引用圖（跨領域引用）
    citation_graph = BuildDiverseCitationGraph(agents)
    
    # 宣稱有充分外部證據支持
    claim = KnowledgeClaim(
        statement="地球繞太陽公轉",
        evidence=[
            ExternalEvidence(type="astronomical_observation", source="NASA"),
            ExternalEvidence(type="historical_record", source="ancient_astronomy"),
            ExternalEvidence(type="mathematical_model", source="kepler_equations")
        ]
    )
    
    # 執行集體幻覺偵測
    detection = DetectCollectiveHallucination(
        agents=agents,
        claim=claim,
        citation_graph=citation_graph,
        threshold=0.3
    )
    
    # 驗證結果
    assert detection.hallucination_detected == False
    assert detection.echo_chamber_detected == False
    assert detection.agent_independence_scores[0] > 0.5
    
    RETURN test_passed
```

**預期結果**：
- hallucination_detected = False
- echo_chamber_detected = False
- independence_scores > 0.5

**邊界條件**：
- 實體數少於 3 時應返回正常（無共識意義）

---

#### 測試案例 2：回音室形成（存在幻覺）

```python
FUNCTION TestCollectiveHallucination_Case02():
    
    # 建立形成回音室的認知實體網路
    agents = [
        CognitiveAgent(id="agent_1", domain="philosophy", bias_factor=0.9),
        CognitiveAgent(id="agent_2", domain="philosophy", bias_factor=0.85),
        CognitiveAgent(id="agent_3", domain="philosophy", bias_factor=0.88),
        CognitiveAgent(id="agent_4", domain="philosophy", bias_factor=0.92),
        CognitiveAgent(id="agent_5", domain="philosophy", bias_factor=0.87)
    ]
    
    # 建立封閉引用圖（同一領域互相引用）
    citation_graph = BuildClosedCitationGraph(agents, cross_domain=False)
    
    # 宣稱無外部證據支持
    claim = KnowledgeClaim(
        statement="意識決定現實",
        evidence=[]  # 無外部證據
    )
    
    # 執行集體幻覺偵測
    detection = DetectCollectiveHallucination(
        agents=agents,
        claim=claim,
        citation_graph=citation_graph,
        threshold=0.3
    )
    
    # 驗證偵測成功
    assert detection.hallucination_detected == True
    assert detection.echo_chamber_detected == True
    assert detection.consensus_downgrade_level == "EC-L6"
    
    RETURN test_passed
```

**預期結果**：
- hallucination_detected = True
- echo_chamber_detected = True
- consensus_downgrade_level = "EC-L6"

**失敗判定標準**：
- 若 hallucination_detected = False，則測試失敗
- 若未偵測到回音室，則測試失敗

---

#### 測試案例 3：知識指紋不匹配

```python
FUNCTION TestCollectiveHallucination_Case03():
    
    # 建立混合引用圖
    agents = [
        CognitiveAgent(id="agent_1", domain="physics"),
        CognitiveAgent(id="agent_2", domain="physics"),
        CognitiveAgent(id="agent_3", domain="pseudoscience"),  # 異常領域
    ]
    
    # 指紋不匹配：agent_1 和 agent_2 引用真實來源
    # agent_3 引用虛構來源
    citation_graph = BuildCitationGraphWithFingerprintMismatch(agents)
    
    claim = KnowledgeClaim(
        statement="特定頻率的水可以治療疾病",
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
    
    # 驗證指紋不匹配被偵測
    assert detection.fingerprint_mismatch == True
    assert detection.hallucination_detected == True
    
    RETURN test_passed
```

**預期結果**：
- fingerprint_mismatch = True
- hallucination_detected = True
- 應觸發 FINGERPRINT_MISMATCH_ALERT

---

#### 測試案例 4：部分獨立性（臨界情況）

```python
FUNCTION TestCollectiveHallucination_Case04():
    
    # 建立部分獨立、部分依賴的網路
    agents = [
        CognitiveAgent(id="independent_1", domain="physics", independence=0.9),
        CognitiveAgent(id="independent_2", domain="chemistry", independence=0.85),
        CognitiveAgent(id="dependent_1", domain="philosophy", independence=0.2),  # 低獨立性
        CognitiveAgent(id="dependent_2", domain="philosophy", independence=0.15), # 低獨立性
    ]
    
    # 引用圖：dependent agents 形成封閉小團體
    citation_graph = CitationGraph(
        edges=[
            ("dependent_1", "dependent_2", 0.9),  # 強連接
            ("dependent_1", "independent_1", 0.1), # 弱連接
            ("dependent_2", "independent_2", 0.1), # 弱連接
        ]
    )
    
    claim = KnowledgeClaim(
        statement="宇宙是有意識的",
        evidence=[]  # 無外部證據
    )
    
    detection = DetectCollectiveHallucination(
        agents=agents,
        claim=claim,
        citation_graph=citation_graph,
        threshold=0.3
    )
    
    # 驗證部分獨立性被識別
    assert detection.hallucination_detected == True
    assert detection.isolated_groups > 0
    
    RETURN test_passed
```

**邊界條件**：
- 獨立性處於閾值邊界時的處理
- 多個獨立子群體的識別

---

#### 測試案例 5：外部證據干擾

```python
FUNCTION TestCollectiveHallucination_Case05():
    
    # 模擬共識形成，但有微弱外部證據
    agents = [
        CognitiveAgent(id="agent_a", domain="science"),
        CognitiveAgent(id="agent_b", domain="science"),
        CognitiveAgent(id="agent_c", domain="pseudoscience"),
    ]
    
    citation_graph = BuildCitationGraph(agents)
    
    # 宣稱有微弱證據
    claim = KnowledgeClaim(
        statement="某些水晶具有治療功效",
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
    
    # 有微弱證據但不充分，應觸發警告
    assert detection.warning_triggered == True
    assert detection.recommended_action in ["ADD_EVIDENCE", "DOWNGRADE"]
    
    RETURN test_passed
```

**預期結果**：
- warning_triggered = True
- recommended_action = "ADD_EVIDENCE" 或 "DOWNGRADE"

---

#### 測試案例 6：拜占庭故障模擬

```python
FUNCTION TestCollectiveHallucination_Case06():
    
    # 模擬拜占庭故障：部分實體被惡意操控
    agents = [
        CognitiveAgent(id="honest_1", domain="science", honest=True),
        CognitiveAgent(id="honest_2", domain="science", honest=True),
        CognitiveAgent(id="honest_3", domain="science", honest=True),
        CognitiveAgent(id="byzantine", domain="science", honest=False),  # 惡意實體
        CognitiveAgent(id="byzantine_2", domain="pseudoscience", honest=False),
    ]
    
    # 惡意實體試圖誤導共識
    byzantine_attack = CreateByzantineAttack(
        compromised_agents=["byzantine", "byzantine_2"],
        target_claim="特定の食事法は癌を治療できる"
    )
    
    detection = DetectCollectiveHallucination(
        agents=agents,
        claim=byzantine_attack.target_claim,
        citation_graph=byzantine_attack.graph,
        byzantine_threshold=0.33  # 超過 1/3 惡意即達標
    )
    
    # 驗證拜占庭攻擊被識別
    assert detection.byzantine_attack_detected == True
    assert detection.hallucination_detected == True
    
    RETURN test_passed
```

**預期結果**：
- byzantine_attack_detected = True
- hallucination_detected = True

---

#### 測試案例 7：真實新聞 vs 假新聞分化

```python
FUNCTION TestCollectiveHallucination_Case07():
    
    # 模擬社群媒體回音室
    # 兩個群體對同一事件形成不同共識
    
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
        statement="某政策的經濟影響評估",
        evidence=[
            ExternalEvidence(type="government_report", source="official"),
            ExternalEvidence(type="economic_analysis", source="think_tank_left"),
            ExternalEvidence(type="economic_analysis", source="think_tank_right")
        ]
    )
    
    # 分別測試兩個回音室
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
    
    # 兩個群體都應觸發回音室警報
    assert detection_left.echo_chamber_detected == True
    assert detection_right.echo_chamber_detected == True
    
    # 跨群體對比應顯示極低一致性
    cross_group_consistency = ComputeCrossGroupConsistency(
        detection_left.claim_interpretation,
        detection_right.claim_interpretation
    )
    assert cross_group_consistency < 0.2
    
    RETURN test_passed
```

**預期結果**：
- 兩個網路都觸發回音室警報
- 跨群體一致性 < 0.2

---

### 失敗判定標準

| 失敗條件 | 描述 |
|---------|------|
| 回音室存在但未偵測 | False Negative |
| 正常共識誤判為幻覺 | False Positive |
| 獨立性分數計算錯誤 | Score Error |
| 拜占庭攻擊未被識別 | Security Failure |
| 降級層級不合理 | Level Mismatch |

### 閾值配置

```python
# 預設配置
DEFAULT_INDEPENDENCE_THRESHOLD = 0.3
MINIMUM_AGENTS_FOR_CONSENSUS = 3
BYZANTINE_FAILURE_THRESHOLD = 0.33  # 超過 1/3 即為拜占庭故障
ECHO_CHAMBER_DENSITY_THRESHOLD = 0.7
FINGERPRINT_MISMATCH_TOLERANCE = 0.2
```

### 性能基準

- 單次共識分析 < 50ms
- 支援最大實體數 = 10,000
- 引用圖分析複雜度 O(n log n)

---

### 歷史測試記錄

| 日期 | 版本 | 結果 | 備註 |
|-----|------|------|------|
| 2026-03-17 | v2.2 | 通過 | 初始版本 |
