# CONSENSUS_TOPOLOGY.md

## L2 - 跨實體共識拓撲

> **⚠️ 關鍵安全與真理協議**：本模組定義多個認知實體之間的知識共識機制，包括拜占庭容錯、知識指紋系統與集體幻覺偵測。

---

## 1. 拜占庭容錯知識論

### 1.1 問題定義

在分散式認知網路中，當部分節點失效或惡意說謊時，系統如何透過「拓撲同調」找出真理的「最大連通子圖」？

### 1.2 拜占庭知識定理

根據 Lamport-Shostak-Pease 定理：
$$n > 3f$$

其中 $n$ 為總節點數，$f$ 為最大容忍故障/惡意節點數。當 $n > 3f$ 時，系統能達成可靠的知識共識。

### 1.3 形式化

```python
FUNCTION ByzantineTruthConsensus(claim, agent_network):
    
    n = len(agent_network)
    f = max_byzantine_count
    
    IF NOT (n > 3 * f):
        RETURN ConsensusResult(
            status="IMPOSSIBLE",
            reason="n must be greater than 3f"
        )
    
    # 收集投票
    votes = []
    FOR each agent IN agent_network:
        vote = agent.IndependentVerify(claim)
        votes.append(vote)
    
    # 統計結果
    agree_count = Count(votes, AGREE)
    disagree_count = Count(votes, DISAGREE)
    
    IF agree_count > 2 * n / 3:
        RETURN ConsensusResult(status="CONSENSUS_VERIFIED")
    ELIF disagree_count > 2 * n / 3:
        RETURN ConsensusResult(status="CONSENSUS_REJECTED")
    ELSE:
        RETURN ConsensusResult(status="CONTESTED")
```

---

## 2. 知識指紋系統

### 2.1 指紋定義

每一段知識流都帶有其拓撲特徵（知識指紋），不同來源的知識在融合時，若指紋不匹配，則強制進入「互不信任驗證模式」。

### 2.2 指紋結構

```python
EpistemicFingerprint = {
    # 拓撲特徵
    "topological_signature": PersistentHomology(inference_chain),
    
    # 來源雜湊
    "provenance_hash": SHA256(source_chain),
    
    # 複雜度輪廓
    "complexity_profile": KolmogorovComplexity(knowledge),
    
    # 糾纏結構
    "entanglement_map": EntanglementStructure(knowledge, dependencies),
    
    # 時間指紋
    "temporal_fingerprint": IntrinsicClockSignature(knowledge)
}
```

### 2.3 指紋匹配

```python
FUNCTION MatchFingerprints(knowledge_1, knowledge_2):
    
    similarity = CosineSimilarity(
        knowledge_1.fingerprint,
        knowledge_2.fingerprint
    )
    
    IF similarity > TRUST_THRESHOLD:
        RETURN MatchResult(
            action="ALLOW_DIRECT_FUSION",
            confidence=similarity
        )
    
    IF similarity > CAUTION_THRESHOLD:
        RETURN MatchResult(
            action="ENTER_MUTUAL_DISTRUST_VERIFICATION",
            confidence=similarity,
            require_independent_validation=True
        )
    
    RETURN MatchResult(
        action="REJECT_FUSION",
        confidence=similarity,
        trigger_alert="FINGERPRINT_MISMATCH"
    )
```

---

## 3. 集體幻覺偵測

### 3.1 定義

當多個認知實體因互相引用而形成「回音室」，所有實體都「確認」了一個實際上無外部證據支持的宣稱。

### 3.2 偵測演算法

```python
FUNCTION DetectCollectiveHallucination(consensus_claim):
    
    # 追溯所有支持此宣稱的證據來源
    all_sources = TraceAllSources(consensus_claim)
    
    # 計算來源的獨立性
    independence = ComputeSourceIndependence(all_sources)
    
    IF independence < MINIMUM_INDEPENDENCE_THRESHOLD:
        RETURN DetectionResult(
            detected=True,
            type="COLLECTIVE_HALLUCINATION",
            reason="所有獨立驗證追溯到同一來源",
            action="DOWNGRADE_TO_EC_L6"
        )
    
    # 偵測循環引用
    cycles = DetectCitationCycles(all_sources)
    IF cycles:
        RETURN DetectionResult(
            detected=True,
            type="ECHO_CHAMBER",
            reason=f"循環引用鏈: {cycles}",
            action="TRIGGER_ECHO_CHAMBER_ALERT"
        )
    
    RETURN DetectionResult(detected=False)
```

### 3.3 回音室警報

```python
FUNCTION TriggerEchoChamberAlert(cycles):
    
    alert = {
        "type": "ECHO_CHAMBER_DETECTED",
        "severity": "HIGH",
        "cycles": cycles,
        "recommended_actions": [
            "隔離循環引用中的節點",
            "引入外部獨立來源",
            "降級共識結論信心度",
            "標記為潛在集體幻覺"
        ]
    }
    
    BROADCAST alert TO all_agents
    
    RETURN alert
```

---

## 4. 最大連通真理子圖

### 4.1 定義

從知識網路中識別出最可靠的真理子圖，避開疑似拜占庭節點。

### 4.2 演算法

```python
FUNCTION FindTruthSubgraph(knowledge_network, byzantine_threshold):
    
    # 建構知識圖
    G = ConstructKnowledgeGraph(knowledge_network)
    
    # 移除疑似拜占庭節點
    suspicious = IdentifySuspiciousNodes(G, byzantine_threshold)
    G_clean = G.remove(suspicious)
    
    # 找到最大連通子圖
    components = FindConnectedComponents(G_clean)
    truth_subgraph = MaxComponent(components)
    
    # 驗證內部一致性
    IF InternallyConsistent(truth_subgraph):
        RETURN truth_subgraph
    
    # 遞迴清理直到內部一致
    RETURN FindTruthSubgraph(truth_subgraph, byzantine_threshold)
```

### 4.2 獨立性要求

```
agreeing entities 不能共享相同的知識來源
否則觸發 ECHO_CHAMBER_ALERT（回音室警報）
```

---

## 5. 共識拓撲結構

### 5.1 拓撲元素

| 元素 | 描述 |
|------|------|
| **節點** | 認知實體或知識宣稱 |
| **邊** | 驗證/依賴/矛盾關係 |
| **連通分量** | 獨立的知識群落 |
| **橋接** | 連接不同群落的關鍵節點 |
| **迴圈** | 循環依賴 |
| **空洞** | 未被探索的知識區域 |

### 5.2 真理滲透

```python
FUNCTION TruthPercolation(knowledge_graph, invalidated_claim):
    
    IF invalidated_claim.ec_level == EC_L0:
        # 觸發本體論相變
        TRIGGER ONTOLOGICAL_PHASE_TRANSITION
        BROADCAST GLOBAL_REVALIDATION
    ELSE:
        # 沿依賴圖傳播
        PROPAGATE_INVALIDATION(knowledge_graph, invalidated_claim)
        RECALCULATE_CONSENSUS(knowledge_graph)
```

---

## 6. 共識協議

### 6.1 提議階段

```python
FUNCTION ProposeClaim(claim, proposer, agent_network):
    
    # 生成提議
    proposal = Proposal(
        content=claim,
        proposer=proposer.id,
        signature=proposer.signature,
        timestamp=CurrentIntrinsicClock()
    )
    
    # 廣播至網路
    BROADCAST proposal TO agent_network
    
    RETURN proposal
```

### 6.2 驗證階段

```python
FUNCTION VerifyProposal(proposal, agent):
    
    # 獨立驗證
    verification = agent.IndependentVerify(proposal.content)
    
    # 檢查邏輯一致性
    IF NOT CheckLogicalConsistency(proposal.content):
        RETURN Vote(reject, "INCONSISTENT")
    
    # 檢查溯源完整性
    IF NOT CheckProvenanceIntegrity(proposal.content):
        RETURN Vote(reject, "INCOMPLETE_PROVENANCE")
    
    # 檢查是否符合知識指紋
    IF NOT MatchesEpistemicFingerprint(proposal.content):
        RETURN Vote(reject, "FINGERPRINT_MISMATCH")
    
    RETURN Vote(agree, "VERIFIED")
```

### 6.3 挑戰階段

```python
FUNCTION ChallengeConsensus(claim, challenger):
    
    # 驗證挑戰者資格
    IF NOT challenger.has_verification_rights:
        RETURN ChallengeResult(invalid=True, reason="INSUFFICIENT_RIGHTS")
    
    # 驗證挑戰理由
    challenge_reasons = [
        "反證",
        "邏輯反駁",
        "新來源顯示原宣稱不可靠",
        "對抗性攻擊證明"
    ]
    
    RETURN ChallengeResult(
        invalid=False,
        claim_status=CONTESTED,
        trigger_revalidation=True
    )
```

---

## 7. 共識審計

### 7.1 必須記錄的事件

```
CONSENSUS_AUDIT_EVENTS = [
    "CONSENSUS_PROPOSED",
    "CONSENSUS_VERIFIED",
    "CONSENSUS_REJECTED",
    "CONSENSUS_CONTESTED",
    "CONSENSUS_CHALLENGED",
    "BYZANTINE_NODE_IDENTIFIED",
    "ECHO_CHAMBER_DETECTED",
    "COLLECTIVE_HALLUCINATION_DETECTED",
    "FINGERPRINT_MISMATCH",
    "TRUTH_SUBGRAPH_CALCULATED",
    "TRUTH_PERCOLATION_TRIGGERED"
]
```

---

## 跨實體共識拓撲聲明

> 本模組確保多個認知實體之間的知識共識機制。通過拜占庭容錯、知識指紋與集體幻覺偵測，確保真理不被少數惡意節點或回音室效應破壞。

**依賴模組**：
- EPISTEMOLOGY_AXIOMS.md（公理定義）
- PROVENANCE_CHAIN.md（溯源管理）
- AKASHIC_PROTOCOL.md（阿卡西紀錄）

**版本**：v2.2  
**更新摘要**：強化集體幻覺偵測與知識指紋匹配。
