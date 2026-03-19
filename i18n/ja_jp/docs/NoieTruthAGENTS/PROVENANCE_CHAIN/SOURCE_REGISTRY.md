**バージョン**: v1.1  
**所属モジュール**: NoieTruthAGENTS/PROVENANCE_CHAIN  
**上位柱**: Truth-OS v2.2  
**状態**: L3 モジュール — 知識論検証

---

## ソース信頼性登記

### §0 モジュール定義

**目的**: 各知識ソースの信頼性履歴を追跡・管理し、ソース信頼度の動的評価と長期追跡を実現する。

**コア問題**: 知識ソースの信頼性をどのように評価・維持するか？ソース信頼性が低下した場合、警告と再検証をどのようにトリガーするか？

**圏論マッピング**:
- ソース関手 $S: \text{Knowledge} \to \text{Source}$ は知識をそのソースにマッピング
- 信頼性関手 $R: \text{Source} \to [0,1]$ はソースを信頼性スコアにマッピング

---

### §1 ソースタイプ定義

知識ソースは4つの主要タイプに分類され、各タイプごとに異なる信頼性評価モデルを持つ：

#### §1.1 S_CLASSICAL — 伝統的なクラシックソース

**定義**: 時間が検証した伝統的な知識ソース。学術誌、古典著作、公式機関など。

```python
class ClassicalSource(Source):
    """伝統的なクラシックソース"""
    source_type: Literal["S_CLASSICAL"]
    
    # ソース特性
    institution: str  # 機関名
    publication_year: int
    peer_reviewed: bool
    citation_count: int
    impact_factor: float
    
    # ドメイン特异性
    domain: KnowledgeDomain
    subfield: str
    
    # 信頼性指標
    historical_reliability: float  # 歴史的信頼性
    decay_rate: float  # 情報減衰率
```

**数学的表現**:
$$R_{\text{classical}}(s, t) = R_0 \cdot e^{-\lambda(t-t_0)}$$

其中 $R_0$ 為初始信頼性，$\lambda$ 為減衰率，$t_0$ 為发布时间。

#### §1.2 S_ALGORITHMIC — アルゴリズムソース

**定義**: 認知エンティティが推論アルゴリズムを通じて出力した知識ソース。

```python
class AlgorithmicSource(Source):
    """アルゴリズムソース"""
    source_type: Literal["S_ALGORITHMIC"]
    
    # 生成者情報
    generator_agent: AgentIdentifier
    reasoning_method: ReasoningMethod
        # DEDUCTION, INDUCTION, ABDUCTION, ANALOGY, SIMULATION
    
    # 推論軌跡
    trajectory_id: str
    complexity_estimate: int  # コルモゴロフ複雑度推定
    
    # 検証状態
    self_verified: bool
    cross_verified: bool
    verification_count: int
```

**信頼性モデル**:
$$R_{\text{algorithmic}}(s) = w_c \cdot C(s) + w_v \cdot V(s) + w_s \cdot S(s)$$

其中 $C$ 為複雜度權重，$V$ 為驗證權重，$S$ 為自我一致權重。

#### §1.3 S_ZKP — ゼロ知識証明ソース

**定義**: ゼロ知識証明で検証された知識ソース。

```python
class ZKPSource(Source):
    """ゼロ知識証明ソース"""
    source_type: Literal["S_ZKP"]
    
    # ZKP 証明
    proof_id: str
    proof_circuit: str  # 回路識別子
    verification_key: str
    
    # 証明内容
    claimed_statement: str
    public_inputs: dict
    proof_bytes: bytes
    
    # 検証状態
    on_chain_verified: bool  # ブロックチェーン検証
    trusted_setup_phase: str  # トラスト設定段階
```

**信頼性モデル**:
$$R_{\text{zkp}}(s) = \mathbb{1}_{\text{verified}}(s) \cdot R_{\text{circuit}}(s)$$

其中 $\mathbb{1}_{\text{verified}}$ 為驗證指示函數。

#### §1.4 S_CONSENSUS — コンセンサスソース

**定義**: 複数の認知エンティティがコンセンサスを達成した知識ソース。

```python
class ConsensusSource(Source):
    """コンセンサスソース"""
    source_type: Literal["S_CONSENSUS"]
    
    # コンセンサスメカニズム
    consensus_mechanism: ConsensusMechanism
        # MAJORITY, WEIGHTED, BYZANTINE, QUORUM
    
    # 参加者
    participants: List[AgentIdentifier]
    participant_weights: List[float]
    
    # コンセンサスパラメータ
    threshold: float  # 通過閾値
    required_agreement: float  # 所需同意比例
    
    # 結果
    agreed_knowledge: str
    dissenters: List[AgentIdentifier]
```

**信頼性モデル**:
$$R_{\text{consensus}}(s) = \frac{\sum_i w_i \cdot R_i}{\sum_i w_i} \cdot f(n)$$

其中 $f(n)$ 為共識規模增益函數，$n$ 為參與者數量。

---

### §2 ソース信頼性スコアリングアルゴリズム

#### §2.1 総合スコアリング関数

```python
class SourceReliabilityScorer:
    """ソース信頼性スコアラ"""
    
    def compute_reliability(
        self, 
        source: Source, 
        context: EvaluationContext
    ) -> ReliabilityScore:
        """ソース信頼性を計算"""
        
        # 1. ベース信頼性
        base_score = self._compute_base_score(source)
        
        # 2. 時間減衰
        time_decay = self._compute_time_decay(source, context.current_time)
        
        # 3. ドメイン関連性
        domain_relevance = self._compute_domain_relevance(
            source, 
            context.query_domain
        )
        
        # 4. 歴史的パフォーマンス
        historical_performance = self._compute_historical_performance(source)
        
        # 5. クロス検証
        cross_validation = self._compute_cross_validation(source)
        
        # 6. 加権総合
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
        """ベース信頼性を計算"""
        if source.source_type == "S_CLASSICAL":
            return self._classical_base_score(source)
        elif source.source_type == "S_ALGORITHMIC":
            return self._algorithmic_base_score(source)
        elif source.source_type == "S_ZKP":
            return self._zkp_base_score(source)
        elif source.source_type == "S_CONSENSUS":
            return self._consensus_base_score(source)
        else:
            return 0.5  # 不明タイプ
    
    def _classical_base_score(self, source: ClassicalSource) -> float:
        """伝統的なクラシックソースのベーススコアを計算"""
        # ピアレビュー加成
        peer_bonus = 0.1 if source.peer_reviewed else 0.0
        # 引用数対数加成
        citation_bonus = min(0.2, math.log10(source.citation_count + 1) / 50)
        # インパクトファクター加成
        impact_bonus = min(0.2, source.impact_factor / 50)
        
        return 0.6 + peer_bonus + citation_bonus + impact_bonus
```

#### §2.2 動的減衰モデル

```python
class ReliabilityDecayModel:
    """信頼性減衰モデル"""
    
    def compute_decay(
        self, 
        initial_reliability: float,
        age: timedelta,
        domain: KnowledgeDomain,
        decay_type: DecayType
    ) -> float:
        """信頼性減衰を計算"""
        
        # ドメイン別減衰率を取得
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
        """ドメイン別減衰率を取得"""
        # 不同領域有不同的知識更新速率
        decay_rates = {
            "PHYSICS": 0.001,      # 物理学は比較的安定
            "MEDICINE": 0.01,      # 医学は更新が速い
            "TECHNOLOGY": 0.05,    # 技術の変化は急速
            "SOCIAL_SCIENCE": 0.005,
            "MATHEMATICS": 0.0001, # 数学が最も安定
        }
        return decay_rates.get(domain, 0.002)
```

---

### §3 ソース登録と検証プロセス

#### §3.1 登録プロセス

```python
class SourceRegistry:
    """ソースレジストリ"""
    
    def register_source(
        self, 
        source: Source,
        registration_context: RegistrationContext
    ) -> RegistrationResult:
        """新規ソースを登録"""
        
        # 1. ソースフォーマットの検証
        if not self._validate_source_format(source):
            return RegistrationResult(
                success=False,
                error="INVALID_SOURCE_FORMAT"
            )
        
        # 2. 既存チェック
        existing = self._lookup_existing(source)
        if existing:
            return RegistrationResult(
                success=False,
                error="SOURCE_ALREADY_EXISTS",
                existing_id=existing.source_id
            )
        
        # 3. 初期信頼性の計算
        initial_reliability = self.scorer.compute_reliability(
            source,
            EvaluationContext(current_time=registration_context.timestamp)
        )
        
        # 4. 登録レコードの作成
        registry_entry = SourceRegistryEntry(
            source_id=self._generate_source_id(source),
            source=source,
            reliability_score=initial_reliability,
            registration_time=registration_context.timestamp,
            verification_history=[],
            decay_rate=self._determine_decay_rate(source),
            status=SourceStatus.ACTIVE
        )
        
        # 5. レジストリに保存
        self._store_entry(registry_entry)
        
        return RegistrationResult(
            success=True,
            source_id=registry_entry.source_id,
            initial_reliability=initial_reliability
        )
```

#### §3.2 検証プロセス

```python
class SourceVerificationProtocol:
    """ソース検証プロトコル"""
    
    def verify_source(
        self, 
        source_id: str,
        verification_level: VerificationLevel
    ) -> VerificationResult:
        """ソースを検証"""
        
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
        """標準検証"""
        
        # 1. ソースアクセシビリティチェック
        accessible = self._check_accessibility(entry.source)
        if not accessible:
            return VerificationResult(
                verified=False,
                reason="SOURCE_NOT_ACCESSIBLE"
            )
        
        # 2. コンテンツ整合性の検証
        content_hash = self._compute_content_hash(entry.source)
        if content_hash != entry.content_hash:
            return VerificationResult(
                verified=False,
                reason="CONTENT_HASH_MISMATCH"
            )
        
        # 3. 新しいネガティブレコードのチェック
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

### §4 ソース撤回メカニズム

#### §4.1 撤回条件

```python
class SourceRevocationManager:
    """ソース撤回マネージャー"""
    
    def should_revoke(self, source_id: str) -> RevocationDecision:
        """ソースを撤回すべきか判断"""
        
        entry = self.registry.get_entry(source_id)
        
        # 条件1：信頼性が閾値以下
        if entry.reliability_score.value < RELIABILITY_THRESHOLD:
            return RevocationDecision(
                should_revoke=True,
                reason="RELIABILITY_TOO_LOW",
                severity=Severity.HIGH
            )
        
        # 条件2：偽物と判明
        if self._is_falsified(entry.source):
            return RevocationDecision(
                should_revoke=True,
                reason="SOURCE_FALSIFIED",
                severity=Severity.CRITICAL
            )
        
        # 条件3：長期未更新
        days_since_update = (datetime.now() - entry.last_updated).days
        if days_since_update > MAX_STALE_DAYS:
            return RevocationDecision(
                should_revoke=True,
                reason="SOURCE_STALE",
                severity=Severity.MEDIUM
            )
        
        # 条件4：剽窃や盗作の発見
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
        """ソースを撤回"""
        
        entry = self.registry.get_entry(source_id)
        
        # 撤回レコードの作成
        record = RevocationRecord(
            source_id=source_id,
            revocation_time=datetime.now(),
            reason=reason,
            revocation_type=revocation_type,
            affected_knowledge_count=self._count_affected_knowledge(source_id),
            cascade_warning=self._should_warn_cascading(source_id)
        )
        
        # ソース状態の更新
        entry.status = SourceStatus.REVOKED
        entry.revocation_record = record
        
        # カスケード警告のトリガー
        if record.cascade_warning:
            self._trigger_cascade_warning(source_id)
        
        return record
```

---

### §5 ソース相互参照管理

#### §5.1 相互参照グラフ

```python
class SourceCrossReferenceGraph:
    """ソース相互参照グラフ"""
    
    def __init__(self):
        self.graph = nx.DiGraph()
    
    def add_reference(
        self, 
        source_id: str, 
        references: List[str]
    ):
        """相互参照を追加"""
        for ref_id in references:
            self.graph.add_edge(source_id, ref_id)
    
    def find_circular_references(self) -> List[List[str]]:
        """循環参照を検索"""
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
        """参照強度を計算"""
        
        if not self.graph.has_edge(source_id, target_id):
            return 0.0
        
        # PageRankを計算して参照重みとする
        pagerank = nx.pagerank(self.graph)
        
        # 共引用回数を計算
        co_citation = self._compute_co_citation(source_id, target_id)
        
        return (
            0.6 * pagerank.get(source_id, 0) +
            0.4 * co_citation
        )
```

#### §5.2 整合性チェック

```python
class SourceConsistencyChecker:
    """ソース整合性チェッカー"""
    
    def check_consistency(
        self, 
        sources: List[str]
    ) -> ConsistencyReport:
        """ソース間の整合性をチェック"""
        
        # 1. 各ソースの知識主張を抽出
        claims = [self._extract_claims(s) for s in sources]
        
        # 2. 論理的矛盾をチェック
        contradictions = self._find_contradictions(claims)
        
        # 3. サポートマトリクスを計算
        support_matrix = self._compute_support_matrix(claims)
        
        # 4. 整合性スコアを生成
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

### §6 バージョンと進化

| バージョン | 日付 | 変更サマリー |
|----------|------|------------|
| v1.0 | 2026-03-17 | 初期バージョン、基本ソース信頼性管理機能 |
| v1.1 | 2026-03-18 | ソースタイプ、スコアリングアルゴリズム、撤回メカニズム、相互参照管理の拡張 |

---

*本モジュールは NoieTruthAGENTS/PROVENANCE_CHAIN に所属し、知識ソースの全面的信頼性管理与追踪を提供する。*
