# SOURCE_REGISTRY.md

**版本**: v1.1  
**所屬模組**: NoieTruthAGENTS/PROVENANCE_CHAIN  
**上級支柱**: Truth-OS v2.2  
**狀態**: L3 模組 — 知識論驗證

---

## 來源可靠度登記

### §0 模組定義

**目的**: 追蹤和管理每個知識來源的可靠度歷史，實現來源可信度的動態評估與長期追蹤。

**核心問題**: 如何評估和維護知識來源的可靠度？當來源可靠度下降時，如何觸發警告和重新驗證？

**範疇論映射**:
- 來源函子 $S: \text{Knowledge} \to \text{Source}$ 將知識映射到其來源
- 可靠度函子 $R: \text{Source} \to [0,1]$ 將來源映射到可靠度分數

---

### §1 來源類型定義

知識來源可分為四大類型，每類型有不同的可靠度評估模型：

#### §1.1 S_CLASSICAL — 傳統經典來源

**定義**: 經過時間檢驗的傳統知識來源，如學術期刊、經典著作、官方機構。

```python
class ClassicalSource(Source):
    """傳統經典來源"""
    source_type: Literal["S_CLASSICAL"]
    
    # 來源特徵
    institution: str  # 機構名稱
    publication_year: int
    peer_reviewed: bool
    citation_count: int
    impact_factor: float
    
    # 領域特異性
    domain: KnowledgeDomain
    subfield: str
    
    # 可靠性指標
    historical_reliability: float  # 歷史可靠度
    decay_rate: float  # 資訊衰減率
```

**數學表示**:
$$R_{\text{classical}}(s, t) = R_0 \cdot e^{-\lambda(t-t_0)}$$

其中 $R_0$ 為初始可靠度，$\lambda$ 為衰減率，$t_0$ 為發布時間。

#### §1.2 S_ALGORITHMIC — 演算法來源

**定義**: 由認知實體通過推理演算法產出的知識來源。

```python
class AlgorithmicSource(Source):
    """演算法來源"""
    source_type: Literal["S_ALGORITHMIC"]
    
    # 產生者資訊
    generator_agent: AgentIdentifier
    reasoning_method: ReasoningMethod
        # DEDUCTION, INDUCTION, ABDUCTION, ANALOGY, SIMULATION
    
    # 推理軌跡
    trajectory_id: str
    complexity_estimate: int  # 柯爾莫哥洛夫複雜度估計
    
    # 驗證狀態
    self_verified: bool
    cross_verified: bool
    verification_count: int
```

**可靠度模型**:
$$R_{\text{algorithmic}}(s) = w_c \cdot C(s) + w_v \cdot V(s) + w_s \cdot S(s)$$

其中 $C$ 為複雜度權重，$V$ 為驗證權重，$S$ 為自我一致權重。

#### §1.3 S_ZKP — 零知識證明來源

**定義**: 經過零知識證明驗證的知識來源。

```python
class ZKPSource(Source):
    """零知識證明來源"""
    source_type: Literal["S_ZKP"]
    
    # ZKP 證明
    proof_id: str
    proof_circuit: str  # 電路識別符
    verification_key: str
    
    # 證明內容
    claimed_statement: str
    public_inputs: dict
    proof_bytes: bytes
    
    # 驗證狀態
    on_chain_verified: bool  # 區塊鏈驗證
    trusted_setup_phase: str  # 信任設置階段
```

**可靠度模型**:
$$R_{\text{zkp}}(s) = \mathbb{1}_{\text{verified}}(s) \cdot R_{\text{circuit}}(s)$$

其中 $\mathbb{1}_{\text{verified}}$ 為驗證指示函數。

#### §1.4 S_CONSENSUS — 共識來源

**定義**: 由多個認知實體達成共識的知識來源。

```python
class ConsensusSource(Source):
    """共識來源"""
    source_type: Literal["S_CONSENSUS"]
    
    # 共識機制
    consensus_mechanism: ConsensusMechanism
        # MAJORITY, WEIGHTED, BYZANTINE, QUORUM
    
    # 參與者
    participants: List[AgentIdentifier]
    participant_weights: List[float]
    
    # 共識參數
    threshold: float  # 通過閾值
    required_agreement: float  # 所需同意比例
    
    # 結果
    agreed_knowledge: str
    dissenters: List[AgentIdentifier]
```

**可靠度模型**:
$$R_{\text{consensus}}(s) = \frac{\sum_i w_i \cdot R_i}{\sum_i w_i} \cdot f(n)$$

其中 $f(n)$ 為共識規模增益函數，$n$ 為參與者數量。

---

### §2 來源可靠性評分演算法

#### §2.1 綜合評分函數

```python
class SourceReliabilityScorer:
    """來源可靠度評分器"""
    
    def compute_reliability(
        self, 
        source: Source, 
        context: EvaluationContext
    ) -> ReliabilityScore:
        """計算來源可靠度"""
        
        # 1. 基礎可靠度
        base_score = self._compute_base_score(source)
        
        # 2. 時間衰減
        time_decay = self._compute_time_decay(source, context.current_time)
        
        # 3. 領域相關性
        domain_relevance = self._compute_domain_relevance(
            source, 
            context.query_domain
        )
        
        # 4. 歷史表現
        historical_performance = self._compute_historical_performance(source)
        
        # 5. 交叉驗證
        cross_validation = self._compute_cross_validation(source)
        
        # 6. 加權綜合
        final_score = (
            w_base * base_score +
            w_time * time_decay +
            w_domain * domain_relevance +
            w_history * historical_performance +
            w_cross * cross_validation
        )
        
        return ReliabilityScore(
            value=final_score,
            confidence=self._compute_confidence(source),
            components={
                "base": base_score,
                "time_decay": time_decay,
                "domain": domain_relevance,
                "history": historical_performance,
                "cross_validation": cross_validation
            }
        )
    
    def _compute_base_score(self, source: Source) -> float:
        """計算基礎可靠度"""
        if source.source_type == "S_CLASSICAL":
            return self._classical_base_score(source)
        elif source.source_type == "S_ALGORITHMIC":
            return self._algorithmic_base_score(source)
        elif source.source_type == "S_ZKP":
            return self._zkp_base_score(source)
        elif source.source_type == "S_CONSENSUS":
            return self._consensus_base_score(source)
        else:
            return 0.5  # 未知類型
    
    def _classical_base_score(self, source: ClassicalSource) -> float:
        """計算傳統來源基礎分數"""
        # 同行評審加成
        peer_bonus = 0.1 if source.peer_reviewed else 0.0
        # 引用數對數加成
        citation_bonus = min(0.2, math.log10(source.citation_count + 1) / 50)
        # 影響因子加成
        impact_bonus = min(0.2, source.impact_factor / 50)
        
        return 0.6 + peer_bonus + citation_bonus + impact_bonus
```

#### §2.2 動態衰減模型

```python
class ReliabilityDecayModel:
    """可靠度衰減模型"""
    
    def compute_decay(
        self, 
        initial_reliability: float,
        age: timedelta,
        domain: KnowledgeDomain,
        decay_type: DecayType
    ) -> float:
        """計算可靠度衰減"""
        
        # 獲取領域特定衰減率
        domain_decay = self._get_domain_decay_rate(domain)
        
        if decay_type == "EXPONENTIAL":
            return initial_reliability * math.exp(-domain_decay * age.days)
        
        elif decay_type == "LOGISTIC":
            k = domain_decay / 100
            return initial_reliability / (1 + math.exp(k * (age.days - 365)))
        
        elif decay_type == "STEPWISE":
            if age.days < 30:
                return initial_reliability
            elif age.days < 365:
                return initial_reliability * 0.9
            elif age.days < 1825:  # 5年
                return initial_reliability * 0.7
            else:
                return initial_reliability * 0.5
        
        else:
            return initial_reliability
    
    def _get_domain_decay_rate(self, domain: KnowledgeDomain) -> float:
        """獲取領域特定衰減率"""
        # 不同領域有不同的知識更新速率
        decay_rates = {
            "PHYSICS": 0.001,      # 物理學相對穩定
            "MEDICINE": 0.01,      # 醫學更新較快
            "TECHNOLOGY": 0.05,    # 技術變化快速
            "SOCIAL_SCIENCE": 0.005,
            "MATHEMATICS": 0.0001, # 數學最穩定
        }
        return decay_rates.get(domain, 0.002)
```

---

### §3 來源註冊與驗證流程

#### §3.1 註冊流程

```python
class SourceRegistry:
    """來源註冊表"""
    
    def register_source(
        self, 
        source: Source,
        registration_context: RegistrationContext
    ) -> RegistrationResult:
        """註冊新來源"""
        
        # 1. 驗證來源格式
        if not self._validate_source_format(source):
            return RegistrationResult(
                success=False,
                error="INVALID_SOURCE_FORMAT"
            )
        
        # 2. 檢查是否已存在
        existing = self._lookup_existing(source)
        if existing:
            return RegistrationResult(
                success=False,
                error="SOURCE_ALREADY_EXISTS",
                existing_id=existing.source_id
            )
        
        # 3. 計算初始可靠度
        initial_reliability = self.scorer.compute_reliability(
            source,
            EvaluationContext(current_time=registration_context.timestamp)
        )
        
        # 4. 創建註冊記錄
        registry_entry = SourceRegistryEntry(
            source_id=self._generate_source_id(source),
            source=source,
            reliability_score=initial_reliability,
            registration_time=registration_context.timestamp,
            verification_history=[],
            decay_rate=self._determine_decay_rate(source),
            status=SourceStatus.ACTIVE
        )
        
        # 5. 存入註冊表
        self._store_entry(registry_entry)
        
        return RegistrationResult(
            success=True,
            source_id=registry_entry.source_id,
            initial_reliability=initial_reliability
        )
```

#### §3.2 驗證流程

```python
class SourceVerificationProtocol:
    """來源驗證協議"""
    
    def verify_source(
        self, 
        source_id: str,
        verification_level: VerificationLevel
    ) -> VerificationResult:
        """驗證來源"""
        
        entry = self.registry.get_entry(source_id)
        
        if verification_level == "MINIMAL":
            return self._verify_minimal(entry)
        elif verification_level == "STANDARD":
            return self._verify_standard(entry)
        elif verification_level == "STRICT":
            return self._verify_strict(entry)
        elif verification_level == "CRYPTOGRAPHIC":
            return self._verify_cryptographic(entry)
    
    def _verify_standard(self, entry: SourceRegistryEntry) -> VerificationResult:
        """標準驗證"""
        
        # 1. 檢查來源可訪問性
        accessible = self._check_accessibility(entry.source)
        if not accessible:
            return VerificationResult(
                verified=False,
                reason="SOURCE_NOT_ACCESSIBLE"
            )
        
        # 2. 驗證內容一致性
        content_hash = self._compute_content_hash(entry.source)
        if content_hash != entry.content_hash:
            return VerificationResult(
                verified=False,
                reason="CONTENT_HASH_MISMATCH"
            )
        
        # 3. 檢查是否有新的負面記錄
        negative_records = self._check_negative_records(entry.source)
        if negative_records:
            return VerificationResult(
                verified=True,
                warnings=["NEGATIVE_RECORDS_FOUND"],
                adjusted_reliability=entry.reliability_score.value * 0.8
            )
        
        return VerificationResult(verified=True)
```

---

### §4 來源撤銷機制

#### §4.1 撤銷條件

```python
class SourceRevocationManager:
    """來源撤銷管理器"""
    
    def should_revoke(self, source_id: str) -> RevocationDecision:
        """判斷是否應該撤銷來源"""
        
        entry = self.registry.get_entry(source_id)
        
        # 條件1：可靠度低於閾值
        if entry.reliability_score.value < RELIABILITY_THRESHOLD:
            return RevocationDecision(
                should_revoke=True,
                reason="RELIABILITY_TOO_LOW",
                severity=Severity.HIGH
            )
        
        # 條件2：被證實為虛假
        if self._is_falsified(entry.source):
            return RevocationDecision(
                should_revoke=True,
                reason="SOURCE_FALSIFIED",
                severity=Severity.CRITICAL
            )
        
        # 條件3：長期未更新
        days_since_update = (datetime.now() - entry.last_updated).days
        if days_since_update > MAX_STALE_DAYS:
            return RevocationDecision(
                should_revoke=True,
                reason="SOURCE_STALE",
                severity=Severity.MEDIUM
            )
        
        # 條件4：發現抄襲或抄襲
        if self._has_plagiarism(entry.source):
            return RevocationDecision(
                should_revoke=True,
                reason="PLAGIARISM_DETECTED",
                severity=Severity.HIGH
            )
        
        return RevocationDecision(should_revoke=False)
    
    def revoke_source(
        self, 
        source_id: str,
        reason: str,
        revocation_type: RevocationType
    ) -> RevocationRecord:
        """撤銷來源"""
        
        entry = self.registry.get_entry(source_id)
        
        # 創建撤銷記錄
        record = RevocationRecord(
            source_id=source_id,
            revocation_time=datetime.now(),
            reason=reason,
            revocation_type=revocation_type,
            affected_knowledge_count=self._count_affected_knowledge(source_id),
            cascade_warning=self._should_warn_cascading(source_id)
        )
        
        # 更新來源狀態
        entry.status = SourceStatus.REVOKED
        entry.revocation_record = record
        
        # 觸發級聯警告
        if record.cascade_warning:
            self._trigger_cascade_warning(source_id)
        
        return record
```

---

### §5 來源交叉引用管理

#### §5.1 交叉引用圖

```python
class SourceCrossReferenceGraph:
    """來源交叉引用圖"""
    
    def __init__(self):
        self.graph = nx.DiGraph()
    
    def add_reference(
        self, 
        source_id: str, 
        references: List[str]
    ):
        """添加交叉引用"""
        for ref_id in references:
            self.graph.add_edge(source_id, ref_id)
    
    def find_circular_references(self) -> List[List[str]]:
        """查找循環引用"""
        try:
            cycles = list(nx.simple_cycles(self.graph))
            return cycles
        except Exception:
            return []
    
    def compute_reference_strength(
        self, 
        source_id: str, 
        target_id: str
    ) -> float:
        """計算引用強度"""
        
        if not self.graph.has_edge(source_id, target_id):
            return 0.0
        
        # 計算 PageRank 作為引用權重
        pagerank = nx.pagerank(self.graph)
        
        # 計算共引次數
        co_citation = self._compute_co_citation(source_id, target_id)
        
        return (
            0.6 * pagerank.get(source_id, 0) +
            0.4 * co_citation
        )
```

#### §5.2 一致性檢查

```python
class SourceConsistencyChecker:
    """來源一致性檢查器"""
    
    def check_consistency(
        self, 
        sources: List[str]
    ) -> ConsistencyReport:
        """檢查來源間的一致性"""
        
        # 1. 提取每個來源的知識主張
        claims = [self._extract_claims(s) for s in sources]
        
        # 2. 檢查邏輯矛盾
        contradictions = self._find_contradictions(claims)
        
        # 3. 計算支援矩陣
        support_matrix = self._compute_support_matrix(claims)
        
        # 4. 生成一致性分數
        consistency_score = self._compute_consistency_score(
            contradictions,
            support_matrix
        )
        
        return ConsistencyReport(
            sources=sources,
            contradictions=contradictions,
            support_matrix=support_matrix,
            consistency_score=consistency_score,
            recommendations=self._generate_recommendations(contradictions)
        )
```

---

### §6 版本與演進

| 版本 | 日期 | 變更摘要 |
|------|------|----------|
| v1.0 | 2026-03-17 | 初始版本，基本來源可靠度管理功能 |
| v1.1 | 2026-03-18 | 擴展來源類型、評分演算法、撤銷機制、交叉引用管理 |

---

*本模組隸屬於 NoieTruthAGENTS/PROVENANCE_CHAIN，提供知識來源的全面可靠度管理與追蹤。*
