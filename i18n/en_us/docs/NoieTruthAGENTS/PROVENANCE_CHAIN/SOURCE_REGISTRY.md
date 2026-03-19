# SOURCE_REGISTRY.md

**Version**: v1.1
**Module**: NoieTruthAGENTS/PROVENANCE_CHAIN
**Parent Framework**: Truth-OS v2.2
**Status**: L3 Module — Epistemological Verification

---

## Source Reliability Registry

### §0 Module Definition

**Purpose**: Track and manage the reliability history of each knowledge source, enabling dynamic assessment and long-term tracking of source credibility.

**Core Question**: How do we evaluate and maintain the reliability of knowledge sources? When source reliability degrades, how do we trigger warnings and re-verification?

**Category Theory Mapping**:
- Source functor $S: \text{Knowledge} \to \text{Source}$ maps knowledge to its source
- Reliability functor $R: \text{Source} \to [0,1]$ maps sources to reliability scores

---

### §1 Source Type Definitions

Knowledge sources can be divided into four major types, each with different reliability assessment models:

#### §1.1 S_CLASSICAL — Traditional Classical Sources

**Definition**: Time-tested traditional knowledge sources such as academic journals, classic works, and official institutions.

```python
class ClassicalSource(Source):
    """Traditional classical source"""
    source_type: Literal["S_CLASSICAL"]
    
    # Source characteristics
    institution: str  # Institution name
    publication_year: int
    peer_reviewed: bool
    citation_count: int
    impact_factor: float
    
    # Domain specificity
    domain: KnowledgeDomain
    subfield: str
    
    # Reliability indicators
    historical_reliability: float  # Historical reliability
    decay_rate: float  # Information decay rate
```

**Mathematical Representation**:
$$R_{\text{classical}}(s, t) = R_0 \cdot e^{-\lambda(t-t_0)}$$

Where $R_0$ is the initial reliability, $\lambda$ is the decay rate, and $t_0$ is the publication time.

#### §1.2 S_ALGORITHMIC — Algorithmic Sources

**Definition**: Knowledge sources produced by cognitive entities through reasoning algorithms.

```python
class AlgorithmicSource(Source):
    """Algorithmic source"""
    source_type: Literal["S_ALGORITHMIC"]
    
    # Generator information
    generator_agent: AgentIdentifier
    reasoning_method: ReasoningMethod
        # DEDUCTION, INDUCTION, ABDUCTION, ANALOGY, SIMULATION
    
    # Reasoning trajectory
    trajectory_id: str
    complexity_estimate: int  # Kolmogorov complexity estimate
    
    # Verification status
    self_verified: bool
    cross_verified: bool
    verification_count: int
```

**Reliability Model**:
$$R_{\text{algorithmic}}(s) = w_c \cdot C(s) + w_v \cdot V(s) + w_s \cdot S(s)$$

Where $C$ is complexity weight, $V$ is verification weight, and $S$ is self-consistency weight.

#### §1.3 S_ZKP — Zero-Knowledge Proof Sources

**Definition**: Knowledge sources verified through zero-knowledge proofs.

```python
class ZKPSource(Source):
    """Zero-knowledge proof source"""
    source_type: Literal["S_ZKP"]
    
    # ZKP proof
    proof_id: str
    proof_circuit: str  # Circuit identifier
    verification_key: str
    
    # Proof content
    claimed_statement: str
    public_inputs: dict
    proof_bytes: bytes
    
    # Verification status
    on_chain_verified: bool  # Blockchain verification
    trusted_setup_phase: str  # Trusted setup phase
```

**Reliability Model**:
$$R_{\text{zkp}}(s) = \mathbb{1}_{\text{verified}}(s) \cdot R_{\text{circuit}}(s)$$

Where $\mathbb{1}_{\text{verified}}$ is the verification indicator function.

#### §1.4 S_CONSENSUS — Consensus Sources

**Definition**: Knowledge sources agreed upon by multiple cognitive entities.

```python
class ConsensusSource(Source):
    """Consensus source"""
    source_type: Literal["S_CONSENSUS"]
    
    # Consensus mechanism
    consensus_mechanism: ConsensusMechanism
        # MAJORITY, WEIGHTED, BYZANTINE, QUORUM
    
    # Participants
    participants: List[AgentIdentifier]
    participant_weights: List[float]
    
    # Consensus parameters
    threshold: float  # Passing threshold
    required_agreement: float  # Required agreement ratio
    
    # Results
    agreed_knowledge: str
    dissenters: List[AgentIdentifier]
```

**Reliability Model**:
$$R_{\text{consensus}}(s) = \frac{\sum_i w_i \cdot R_i}{\sum_i w_i} \cdot f(n)$$

Where $f(n)$ is the consensus scale gain function and $n$ is the number of participants.

---

### §2 Source Reliability Scoring Algorithm

#### §2.1 Comprehensive Scoring Function

```python
class SourceReliabilityScorer:
    """Source reliability scorer"""
    
    def compute_reliability(
        self, 
        source: Source, 
        context: EvaluationContext
    ) -> ReliabilityScore:
        """Compute source reliability"""
        
        # 1. Base reliability
        base_score = self._compute_base_score(source)
        
        # 2. Time decay
        time_decay = self._compute_time_decay(source, context.current_time)
        
        # 3. Domain relevance
        domain_relevance = self._compute_domain_relevance(
            source, 
            context.query_domain
        )
        
        # 4. Historical performance
        historical_performance = self._compute_historical_performance(source)
        
        # 5. Cross-validation
        cross_validation = self._compute_cross_validation(source)
        
        # 6. Weighted combination
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
        """Compute base reliability"""
        if source.source_type == "S_CLASSICAL":
            return self._classical_base_score(source)
        elif source.source_type == "S_ALGORITHMIC":
            return self._algorithmic_base_score(source)
        elif source.source_type == "S_ZKP":
            return self._zkp_base_score(source)
        elif source.source_type == "S_CONSENSUS":
            return self._consensus_base_score(source)
        else:
            return 0.5  # Unknown type
    
    def _classical_base_score(self, source: ClassicalSource) -> float:
        """Compute classical source base score"""
        # Peer review bonus
        peer_bonus = 0.1 if source.peer_reviewed else 0.0
        # Citation count logarithmic bonus
        citation_bonus = min(0.2, math.log10(source.citation_count + 1) / 50)
        # Impact factor bonus
        impact_bonus = min(0.2, source.impact_factor / 50)
        
        return 0.6 + peer_bonus + citation_bonus + impact_bonus
```

#### §2.2 Dynamic Decay Model

```python
class ReliabilityDecayModel:
    """Reliability decay model"""
    
    def compute_decay(
        self, 
        initial_reliability: float,
        age: timedelta,
        domain: KnowledgeDomain,
        decay_type: DecayType
    ) -> float:
        """Compute reliability decay"""
        
        # Get domain-specific decay rate
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
            elif age.days < 1825:  # 5 years
                return initial_reliability * 0.7
            else:
                return initial_reliability * 0.5
        
        else:
            return initial_reliability
    
    def _get_domain_decay_rate(self, domain: KnowledgeDomain) -> float:
        """Get domain-specific decay rate"""
        # Different domains have different knowledge update rates
        decay_rates = {
            "PHYSICS": 0.001,      # Physics is relatively stable
            "MEDICINE": 0.01,      # Medicine updates faster
            "TECHNOLOGY": 0.05,    # Technology changes rapidly
            "SOCIAL_SCIENCE": 0.005,
            "MATHEMATICS": 0.0001, # Mathematics is most stable
        }
        return decay_rates.get(domain, 0.002)
```

---

### §3 Source Registration and Verification Process

#### §3.1 Registration Process

```python
class SourceRegistry:
    """Source registry"""
    
    def register_source(
        self, 
        source: Source,
        registration_context: RegistrationContext
    ) -> RegistrationResult:
        """Register a new source"""
        
        # 1. Validate source format
        if not self._validate_source_format(source):
            return RegistrationResult(
                success=False,
                error="INVALID_SOURCE_FORMAT"
            )
        
        # 2. Check if already exists
        existing = self._lookup_existing(source)
        if existing:
            return RegistrationResult(
                success=False,
                error="SOURCE_ALREADY_EXISTS",
                existing_id=existing.source_id
            )
        
        # 3. Compute initial reliability
        initial_reliability = self.scorer.compute_reliability(
            source,
            EvaluationContext(current_time=registration_context.timestamp)
        )
        
        # 4. Create registration record
        registry_entry = SourceRegistryEntry(
            source_id=self._generate_source_id(source),
            source=source,
            reliability_score=initial_reliability,
            registration_time=registration_context.timestamp,
            verification_history=[],
            decay_rate=self._determine_decay_rate(source),
            status=SourceStatus.ACTIVE
        )
        
        # 5. Store in registry
        self._store_entry(registry_entry)
        
        return RegistrationResult(
            success=True,
            source_id=registry_entry.source_id,
            initial_reliability=initial_reliability
        )
```

#### §3.2 Verification Process

```python
class SourceVerificationProtocol:
    """Source verification protocol"""
    
    def verify_source(
        self, 
        source_id: str,
        verification_level: VerificationLevel
    ) -> VerificationResult:
        """Verify a source"""
        
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
        """Standard verification"""
        
        # 1. Check source accessibility
        accessible = self._check_accessibility(entry.source)
        if not accessible:
            return VerificationResult(
                verified=False,
                reason="SOURCE_NOT_ACCESSIBLE"
            )
        
        # 2. Verify content consistency
        content_hash = self._compute_content_hash(entry.source)
        if content_hash != entry.content_hash:
            return VerificationResult(
                verified=False,
                reason="CONTENT_HASH_MISMATCH"
            )
        
        # 3. Check for new negative records
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

### §4 Source Revocation Mechanism

#### §4.1 Revocation Conditions

```python
class SourceRevocationManager:
    """Source revocation manager"""
    
    def should_revoke(self, source_id: str) -> RevocationDecision:
        """Determine if source should be revoked"""
        
        entry = self.registry.get_entry(source_id)
        
        # Condition 1: Reliability below threshold
        if entry.reliability_score.value < RELIABILITY_THRESHOLD:
            return RevocationDecision(
                should_revoke=True,
                reason="RELIABILITY_TOO_LOW",
                severity=Severity.HIGH
            )
        
        # Condition 2: Confirmed as false
        if self._is_falsified(entry.source):
            return RevocationDecision(
                should_revoke=True,
                reason="SOURCE_FALSIFIED",
                severity=Severity.CRITICAL
            )
        
        # Condition 3: Long time without update
        days_since_update = (datetime.now() - entry.last_updated).days
        if days_since_update > MAX_STALE_DAYS:
            return RevocationDecision(
                should_revoke=True,
                reason="SOURCE_STALE",
                severity=Severity.MEDIUM
            )
        
        # Condition 4: Plagiarism detected
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
        """Revoke a source"""
        
        entry = self.registry.get_entry(source_id)
        
        # Create revocation record
        record = RevocationRecord(
            source_id=source_id,
            revocation_time=datetime.now(),
            reason=reason,
            revocation_type=revocation_type,
            affected_knowledge_count=self._count_affected_knowledge(source_id),
            cascade_warning=self._should_warn_cascading(source_id)
        )
        
        # Update source status
        entry.status = SourceStatus.REVOKED
        entry.revocation_record = record
        
        # Trigger cascade warning
        if record.cascade_warning:
            self._trigger_cascade_warning(source_id)
        
        return record
```

---

### §5 Source Cross-Reference Management

#### §5.1 Cross-Reference Graph

```python
class SourceCrossReferenceGraph:
    """Source cross-reference graph"""
    
    def __init__(self):
        self.graph = nx.DiGraph()
    
    def add_reference(
        self, 
        source_id: str, 
        references: List[str]
    ):
        """Add cross-references"""
        for ref_id in references:
            self.graph.add_edge(source_id, ref_id)
    
    def find_circular_references(self) -> List[List[str]]:
        """Find circular references"""
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
        """Compute reference strength"""
        
        if not self.graph.has_edge(source_id, target_id):
            return 0.0
        
        # Compute PageRank as reference weight
        pagerank = nx.pagerank(self.graph)
        
        # Compute co-citation count
        co_citation = self._compute_co_citation(source_id, target_id)
        
        return (
            0.6 * pagerank.get(source_id, 0) +
            0.4 * co_citation
        )
```

#### §5.2 Consistency Check

```python
class SourceConsistencyChecker:
    """Source consistency checker"""
    
    def check_consistency(
        self, 
        sources: List[str]
    ) -> ConsistencyReport:
        """Check consistency between sources"""
        
        # 1. Extract knowledge claims from each source
        claims = [self._extract_claims(s) for s in sources]
        
        # 2. Check for logical contradictions
        contradictions = self._find_contradictions(claims)
        
        # 3. Compute support matrix
        support_matrix = self._compute_support_matrix(claims)
        
        # 4. Generate consistency score
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

### §6 Version and Evolution

| Version | Date | Change Summary |
|---------|------|---------------|
| v1.0 | 2026-03-17 | Initial version, basic source reliability management |
| v1.1 | 2026-03-18 | Extended source types, scoring algorithms, revocation mechanism, cross-reference management |

---

*This module belongs to NoieTruthAGENTS/PROVENANCE_CHAIN, providing comprehensive reliability management and tracking for knowledge sources.*
