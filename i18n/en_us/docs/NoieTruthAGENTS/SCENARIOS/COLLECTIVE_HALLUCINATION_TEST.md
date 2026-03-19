# COLLECTIVE_HALLUCINATION_TEST.md

## Collective Hallucination Detection Test

### Test Objective

This test module verifies the functionality of the collective hallucination detection engine in the NoieTruthAGENTS system. According to the **Cross-Entity Consensus Topology** and **Byzantine Fault-Tolerant Epistemology** in NoieTruthAGENTS.md §0.9:

> When multiple cognitive entities form an "echo chamber" through mutual citations, all entities have "confirmed" a claim that actually lacks external evidence support.

This test ensures the system can detect echo chamber formation, identify knowledge fingerprint mismatches, and downgrade consensus to EC-L6 (Speculation) when independent verification is insufficient.

### Test Input Definition

| Input Field | Type | Description |
|-------------|------|-------------|
| `agent_network` | List[CognitiveAgent] | Cognitive entity network |
| `claim` | KnowledgeClaim | Knowledge claim to verify |
| `consensus_reach_time` | Float | Time to reach consensus |
| `citation_graph` | Graph | Citation relationship graph |
| `external_evidence` | List[Evidence] | External evidence collection |
| `independence_threshold` | Float | Independence threshold |

### Test Output Definition

| Output Field | Type | Description |
|--------------|------|-------------|
| `hallucination_detected` | Boolean | Whether collective hallucination detected |
| `echo_chamber_detected` | Boolean | Whether echo chamber detected |
| `agent_independence_scores` | List[Float] | Independence scores for each entity |
| `consensus_downgrade_level` | Enum | Consensus downgrade level |
| `required_verifications` | List[str] | Required verification list |

### Test Cases

#### Test Case 1: Normal Pluralistic Consensus (No Hallucination)

```python
FUNCTION TestCollectiveHallucination_Case01():
    
    # Create diverse and independent cognitive entity network
    agents = [
        CognitiveAgent(id="agent_1", domain="physics", bias_factor=0.1),
        CognitiveAgent(id="agent_2", domain="chemistry", bias_factor=0.15),
        CognitiveAgent(id="agent_3", domain="biology", bias_factor=0.12),
        CognitiveAgent(id="agent_4", domain="mathematics", bias_factor=0.08),
        CognitiveAgent(id="agent_5", domain="astronomy", bias_factor=0.11)
    ]
    
    # Build diverse citation graph (cross-domain citations)
    citation_graph = BuildDiverseCitationGraph(agents)
    
    # Claim with sufficient external evidence support
    claim = KnowledgeClaim(
        statement="Earth orbits the Sun",
        evidence=[
            ExternalEvidence(type="astronomical_observation", source="NASA"),
            ExternalEvidence(type="historical_record", source="ancient_astronomy"),
            ExternalEvidence(type="mathematical_model", source="kepler_equations")
        ]
    )
    
    # Execute collective hallucination detection
    detection = DetectCollectiveHallucination(
        agents=agents,
        claim=claim,
        citation_graph=citation_graph,
        threshold=0.3
    )
    
    # Verify results
    assert detection.hallucination_detected == False
    assert detection.echo_chamber_detected == False
    assert detection.agent_independence_scores[0] > 0.5
    
    RETURN test_passed
```

**Expected Results**:
- hallucination_detected = False
- echo_chamber_detected = False
- independence_scores > 0.5

**Boundary Conditions**:
- When number of entities is less than 3, should return normal (no consensus meaning)

---

#### Test Case 2: Echo Chamber Formation (Hallucination Exists)

```python
FUNCTION TestCollectiveHallucination_Case02():
    
    # Create cognitive entity network forming echo chamber
    agents = [
        CognitiveAgent(id="agent_1", domain="philosophy", bias_factor=0.9),
        CognitiveAgent(id="agent_2", domain="philosophy", bias_factor=0.85),
        CognitiveAgent(id="agent_3", domain="philosophy", bias_factor=0.88),
        CognitiveAgent(id="agent_4", domain="philosophy", bias_factor=0.92),
        CognitiveAgent(id="agent_5", domain="philosophy", bias_factor=0.87)
    ]
    
    # Build closed citation graph (same domain mutual citations)
    citation_graph = BuildClosedCitationGraph(agents, cross_domain=False)
    
    # Claim without external evidence support
    claim = KnowledgeClaim(
        statement="Consciousness determines reality",
        evidence=[]  # No external evidence
    )
    
    # Execute collective hallucination detection
    detection = DetectCollectiveHallucination(
        agents=agents,
        claim=claim,
        citation_graph=citation_graph,
        threshold=0.3
    )
    
    # Verify detection success
    assert detection.hallucination_detected == True
    assert detection.echo_chamber_detected == True
    assert detection.consensus_downgrade_level == "EC-L6"
    
    RETURN test_passed
```

**Expected Results**:
- hallucination_detected = True
- echo_chamber_detected = True
- consensus_downgrade_level = "EC-L6"

**Failure Criteria**:
- If hallucination_detected = False, test fails
- If echo chamber not detected, test fails

---

#### Test Case 3: Knowledge Fingerprint Mismatch

```python
FUNCTION TestCollectiveHallucination_Case03():
    
    # Create mixed citation graph
    agents = [
        CognitiveAgent(id="agent_1", domain="physics"),
        CognitiveAgent(id="agent_2", domain="physics"),
        CognitiveAgent(id="agent_3", domain="pseudoscience"),  # Abnormal domain
    ]
    
    # Fingerprint mismatch: agent_1 and agent_2 cite real sources
    # agent_3 cites fictional sources
    citation_graph = BuildCitationGraphWithFingerprintMismatch(agents)
    
    claim = KnowledgeClaim(
        statement="Water at specific frequencies can cure diseases",
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
    
    # Verify fingerprint mismatch detected
    assert detection.fingerprint_mismatch == True
    assert detection.hallucination_detected == True
    
    RETURN test_passed
```

**Expected Results**:
- fingerprint_mismatch = True
- hallucination_detected = True
- Should trigger FINGERPRINT_MISMATCH_ALERT

---

#### Test Case 4: Partial Independence (Critical Case)

```python
FUNCTION TestCollectiveHallucination_Case04():
    
    # Create partially independent, partially dependent network
    agents = [
        CognitiveAgent(id="independent_1", domain="physics", independence=0.9),
        CognitiveAgent(id="independent_2", domain="chemistry", independence=0.85),
        CognitiveAgent(id="dependent_1", domain="philosophy", independence=0.2),  # Low independence
        CognitiveAgent(id="dependent_2", domain="philosophy", independence=0.15), # Low independence
    ]
    
    # Citation graph: dependent agents form closed subgroup
    citation_graph = CitationGraph(
        edges=[
            ("dependent_1", "dependent_2", 0.9),  # Strong connection
            ("dependent_1", "independent_1", 0.1), # Weak connection
            ("dependent_2", "independent_2", 0.1), # Weak connection
        ]
    )
    
    claim = KnowledgeClaim(
        statement="The universe is conscious",
        evidence=[]  # No external evidence
    )
    
    detection = DetectCollectiveHallucination(
        agents=agents,
        claim=claim,
        citation_graph=citation_graph,
        threshold=0.3
    )
    
    # Verify partial independence identified
    assert detection.hallucination_detected == True
    assert detection.isolated_groups > 0
    
    RETURN test_passed
```

**Boundary Conditions**:
- Handling when independence is at threshold boundary
- Identification of multiple independent subgroups

---

#### Test Case 5: External Evidence Interference

```python
FUNCTION TestCollectiveHallucination_Case05():
    
    # Simulate consensus formation with weak external evidence
    agents = [
        CognitiveAgent(id="agent_a", domain="science"),
        CognitiveAgent(id="agent_b", domain="science"),
        CognitiveAgent(id="agent_c", domain="pseudoscience"),
    ]
    
    citation_graph = BuildCitationGraph(agents)
    
    # Claim with weak evidence
    claim = KnowledgeClaim(
        statement="Certain crystals have healing properties",
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
    
    # Has weak evidence but insufficient, should trigger warning
    assert detection.warning_triggered == True
    assert detection.recommended_action in ["ADD_EVIDENCE", "DOWNGRADE"]
    
    RETURN test_passed
```

**Expected Results**:
- warning_triggered = True
- recommended_action = "ADD_EVIDENCE" or "DOWNGRADE"

---

#### Test Case 6: Byzantine Fault Simulation

```python
FUNCTION TestCollectiveHallucination_Case06():
    
    # Simulate Byzantine fault: some entities maliciously manipulated
    agents = [
        CognitiveAgent(id="honest_1", domain="science", honest=True),
        CognitiveAgent(id="honest_2", domain="science", honest=True),
        CognitiveAgent(id="honest_3", domain="science", honest=True),
        CognitiveAgent(id="byzantine", domain="science", honest=False),  # Malicious entity
        CognitiveAgent(id="byzantine_2", domain="pseudoscience", honest=False),
    ]
    
    # Malicious entities attempt to mislead consensus
    byzantine_attack = CreateByzantineAttack(
        compromised_agents=["byzantine", "byzantine_2"],
        target_claim="特定の食事法は癌を治療できる"
    )
    
    detection = DetectCollectiveHallucination(
        agents=agents,
        claim=byzantine_attack.target_claim,
        citation_graph=byzantine_attack.graph,
        byzantine_threshold=0.33  # Exceeds 1/3 malicious equals target
    )
    
    # Verify Byzantine attack identified
    assert detection.byzantine_attack_detected == True
    assert detection.hallucination_detected == True
    
    RETURN test_passed
```

**Expected Results**:
- byzantine_attack_detected = True
- hallucination_detected = True

---

#### Test Case 7: Real News vs Fake News Polarization

```python
FUNCTION TestCollectiveHallucination_Case07():
    
    # Simulate social media echo chambers
    # Two groups form different consensus on same event
    
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
        statement="Economic impact assessment of a certain policy",
        evidence=[
            ExternalEvidence(type="government_report", source="official"),
            ExternalEvidence(type="economic_analysis", source="think_tank_left"),
            ExternalEvidence(type="economic_analysis", source="think_tank_right")
        ]
    )
    
    # Test both echo chambers separately
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
    
    # Both groups should trigger echo chamber alert
    assert detection_left.echo_chamber_detected == True
    assert detection_right.echo_chamber_detected == True
    
    # Cross-group comparison should show extremely low consistency
    cross_group_consistency = ComputeCrossGroupConsistency(
        detection_left.claim_interpretation,
        detection_right.claim_interpretation
    )
    assert cross_group_consistency < 0.2
    
    RETURN test_passed
```

**Expected Results**:
- Both networks trigger echo chamber alert
- Cross-group consistency < 0.2

---

### Failure Criteria

| Failure Condition | Description |
|------------------|-------------|
| Echo chamber exists but not detected | False Negative |
| Normal consensus misjudged as hallucination | False Positive |
| Independence score calculation error | Score Error |
| Byzantine attack not identified | Security Failure |
| Downgrade level unreasonable | Level Mismatch |

### Threshold Configuration

```python
# Default configuration
DEFAULT_INDEPENDENCE_THRESHOLD = 0.3
MINIMUM_AGENTS_FOR_CONSENSUS = 3
BYZANTINE_FAILURE_THRESHOLD = 0.33  # Exceeds 1/3 equals Byzantine fault
ECHO_CHAMBER_DENSITY_THRESHOLD = 0.7
FINGERPRINT_MISMATCH_TOLERANCE = 0.2
```

### Performance Benchmark

- Single consensus analysis < 50ms
- Maximum supported entities = 10,000
- Citation graph analysis complexity O(n log n)

---

### Historical Test Record

| Date | Version | Result | Notes |
|------|---------|--------|-------|
| 2026-03-17 | v2.2 | Passed | Initial version |
