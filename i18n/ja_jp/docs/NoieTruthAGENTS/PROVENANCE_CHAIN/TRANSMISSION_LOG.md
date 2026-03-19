**バージョン**: v1.1  
**所属モジュール**: NoieTruthAGENTS/PROVENANCE_CHAIN  
**上位柱**: Truth-OS v2.2  
**状態**: L3 モジュール — 知識論検証

---

## 知識伝達チェーン記録

### §0 モジュール定義

**目的**: 認知エンティティ間の知識伝達履歴を完全に記録し、知識の遡及可能性と意味整合性を確保する。

**コア問題**: 知識がある認知エンティティから別のエンティティに伝達されるとき、伝達プロセスの整合性、検証可能性、意味的忠実性をどのように確保するか？

**圏論マッピング**: 
- 伝達関手 $T: \text{Cog} \to \text{Cog}$ は知識状態を接收状態にマッピング
- 検証関手 $V: \text{Transmission} \to \text{Verify}$ は伝達の有効性を確認

---

### §1 伝達イベントタイプ

知識伝達には4つの基本的なイベントタイプがある：

#### §1.1 建立イベント (ESTABLISHMENT)

**定義**: ネイティブ知識の初回作成。前置伝達ソースなし。

```python
class EstablishmentEvent(TxEvent):
    """知識のネイティブ作成イベント"""
    event_type: Literal["ESTABLISHMENT"]
    knowledge_id: str
    creator_agent: str
    creation_method: CreationMethod  # DIRECT, DERIVED, INFERRED
    initial_provenance: ProvenanceChain
    intrinsic_clock: IntrinsicTimestamp
```

**数学的表現**:
$$E_{\text{establish}}(k) = (k_{\text{new}}, \text{Creator}(A), t_0, \emptyset)$$

其中 $k_{\text{new}}$ 為新創建的知識，$t_0$ 為創建時間戳，$\emptyset$ 表示無前置來源。

#### §1.2 転送イベント (FORWARDING)

**定義**: 知識がある認知エンティティから別のエンティティに転送。意味内容は変化しない。

```python
class ForwardingEvent(TxEvent):
    """知識の転送イベント、意味整合性を維持"""
    event_type: Literal["FORWARDING"]
    knowledge_id: str
    source_agent: str
    target_agent: str
    transmission_medium: Medium  # DIRECT, BROADCAST, RELAY
    semantic_integrity_hash: str  # SHA-256 of content
    drift_measurement: float = 0.0
```

**数学的表現**:
$$E_{\text{forward}}(k, A \to B) = (k, B, t, \text{hash}(k))$$

整合性條件: $\text{hash}(k_{\text{sent}}) = \text{hash}(k_{\text{received}})$

#### §1.3 変換イベント (TRANSFORMATION)

**定義**: 知識が伝達プロセス中に意味変換を経験（要約、翻訳、圧縮など）。

```python
class TransformationEvent(TxEvent):
    """知識の意味変換イベント"""
    event_type: Literal["TRANSFORMATION"]
    knowledge_id: str
    source_agent: str
    target_agent: str
    transformation_type: TransformationType
        # ABSTRACTION, TRANSLATION, COMPRESSION, ENRICHMENT, DERIVATION
    original_content_hash: str
    transformed_content_hash: str
    transformation_metadata: TransformationMetadata
    semantic_drift_vector: SemanticDriftVector
```

**意味ドリフト度量**:
$$\text{Drift}(k_{\text{orig}}, k_{\text{trans}}) = \frac{d_{\text{semantic}}(k_{\text{orig}}, k_{\text{trans}})}{\|k_{\text{orig}}\|}$$

其中 $d_{\text{semantic}}$ 為語義距離函數。

#### §1.4 受信イベント (RECEPTION)

**定義**: 知識が目標認知エンティティに正常に受信・確認された。

```python
class ReceptionEvent(TxEvent):
    """知識の受信確認イベント"""
    event_type: Literal["RECEPTION"]
    knowledge_id: str
    receiver_agent: str
    source_agent: str
    reception_timestamp: IntrinsicTimestamp
    verification_status: VerificationStatus  # VERIFIED, PARTIAL, FAILED
    integration_result: IntegrationResult
```

---

### §2 伝達イベントフォーマット

各伝達イベントには以下のフィールドがなければならない：

```python
@dataclass
class TransmissionLogEntry:
    """完全な伝達ログエントリ"""
    # 識別子
    entry_id: str  # UUID v7
    knowledge_id: str
    
    # イベント時空
    event_type: TxEventType
    transmission_timestamp: IntrinsicTimestamp
    physical_timestamp: PhysicalTimestamp
    
    # 参加者
    source_agent: AgentIdentifier
    target_agent: AgentIdentifier
    medium: TransmissionMedium
    
    # コンテンツ同定
    content_hash: str  # SHA-3-256
    semantic_integrity_hash: str
    transformation_record: Optional[TransformationRecord]
    
    # 監査軌跡
    audit_signature: str  # Agent's cryptographic signature
    previous_entry_hash: str  # Chain linkage
```

**完全フォーマット数学的表現**:
$$L = (e_{\text{id}}, k_{\text{id}}, \tau_{\text{tx}}, \tau_{\text{phy}}, A_{\text{src}}, A_{\text{tgt}}, h_{\text{cont}}, h_{\text{sem}}, \sigma_{\text{audit}}, h_{\text{prev}})$$

---

### §3 伝達チェーンの検証と監査

#### §3.1 チェーン整合性検証

```python
def verify_transmission_chain(chain: List[TransmissionLogEntry]) -> ChainVerificationResult:
    """伝達チェーンの整合性を検証"""
    
    # 1. 順序検証：タイムスタンプが単調増加するかチェック
    for i in range(1, len(chain)):
        if chain[i].transmission_timestamp <= chain[i-1].transmission_timestamp:
            return ChainVerificationResult(
                valid=False,
                violation=TIMESTAMP_VIOLATION,
                details=f"Timestamp not monotonic at index {i}"
            )
    
    # 2. リンケージ検証：ハッシュリンクをチェック
    for i in range(1, len(chain)):
        expected_prev_hash = chain[i-1].compute_hash()
        if chain[i].previous_entry_hash != expected_prev_hash:
            return ChainVerificationResult(
                valid=False,
                violation=CHAIN_LINK_VIOLATION,
                details=f"Chain link broken at index {i}"
            )
    
    # 3. 意味整合性検証
    for event in chain:
        if event.event_type == "TRANSFORMATION":
            if event.semantic_drift_vector.magnitude > DRIFT_THRESHOLD:
                return ChainVerificationResult(
                    valid=False,
                    violation=SEMANTIC_DRIFT_VIOLATION,
                    details=f"Excessive semantic drift at {event.entry_id}"
                )
    
    return ChainVerificationResult(valid=True)
```

#### §3.2 監査署名プロトコル

各伝達イベントには送信側と受信側の共同署名が必要：

$$ \sigma = \text{Sign}_{sk_{\text{agent}}}(H(L)) $$

其中 $H(L)$ 為日誌條目的密碼學雜湊值。

---

### §4 クロスエージェント伝達プロトコル

#### §4.1 標準伝達フロー

```
┌─────────────────────────────────────────────────────────────────┐
│                    知識伝達プロトコル (KTP v1.1)                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  [Agent A]                              [Agent B]                 │
│     │                                       │                   │
│     │  1. TRANSMISSION_REQUEST             │                   │
│     │──────────────────────────────────────>│                   │
│     │                                       │                   │
│     │  2. TRANSMISSION_ACK + Capabilities   │                   │
│     │<──────────────────────────────────────│                   │
│     │                                       │                   │
│     │  3. Knowledge Payload + Signature     │                   │
│     │──────────────────────────────────────>│                   │
│     │                                       │                   │
│     │  4. Verification + Integration Result │                   │
│     │<──────────────────────────────────────│                   │
│     │                                       │                   │
│     │  5. Final Acknowledgment             │                   │
│     │──────────────────────────────────────>│                   │
│     │                                       │                   │
└─────────────────────────────────────────────────────────────────┘
```

#### §4.2 伝達リクエストフォーマット

```python
@dataclass
class TransmissionRequest:
    """知識伝達リクエスト"""
    request_id: str
    requesting_agent: AgentIdentifier
    knowledge_id: str
    requested_verification_level: VerificationLevel
        # MINIMAL, STANDARD, STRICT, CRYPTOGRAPHIC
    preferred_medium: TransmissionMedium
    semantic_fidelity_requirement: float  # 0.0-1.0
    timeout: timedelta
```

#### §4.3 応答フォーマット

```python
@dataclass
class TransmissionResponse:
    """知識伝達応答"""
    request_id: str
    accepting: bool
    capabilities: AgentCapabilities
    proposed_medium: TransmissionMedium
    verification_commitment: VerificationLevel
    integration_capacity: ResourceCapacity
```

---

### §5 伝達失敗処理メカニズム

#### §5.1 失敗タイプ分類

| 失敗タイプ | 説明 | 処理戦略 |
|----------|------|---------|
| `TIMEOUT` | 伝達タイムアウト | 再試行 + 指数バックオフ |
| `INTEGRITY_VIOLATION` | 整合性破壊 | 拒否 + アラート |
| `SEMANTIC_DRIFT_EXCEEDED` | 意味ドリフト過大 | ネゴシエーション + 降格 |
| `VERIFICATION_FAILED` | 検証失敗 | 隔離 + 監査 |
| `AGENT_UNAVAILABLE` | 対象エージェント利用不可 | ルーティング + 一時保存 |

#### §5.2 再試行戦略

```python
class TransmissionRetryStrategy:
    """伝達再試行戦略"""
    
    def __init__(self):
        self.max_retries = 5
        self.base_delay = timedelta(seconds=1)
        self.exponential_base = 2.0
        self.jitter_factor = 0.1
    
    def compute_delay(self, attempt: int) -> timedelta:
        """指数バックオフ遅延を計算"""
        delay = self.base_delay * (self.exponential_base ** attempt)
        # ランダムジッターを追加
        jitter = delay * self.jitter_factor * random.uniform(-1, 1)
        return delay + jitter
    
    def should_retry(self, failure: TransmissionFailure) -> bool:
        """再試行すべきか判断"""
        if failure.failure_type in [INTEGRITY_VIOLATION, VERIFICATION_FAILED]:
            return False  # これらの失敗は再試行不可
        return True
```

#### §5.3 失敗記録

```python
@dataclass
class TransmissionFailure:
    """伝達失敗記録"""
    failure_id: str
    transmission_entry: TransmissionLogEntry
    failure_type: FailureType
    error_message: str
    attempted_retries: int
    final_state: TransmissionState  # FAILED, RETRYING, ROLLED_BACK
    resolution: Optional[str]
```

---

### §6 伝達ログクエリ API

```python
class TransmissionLogAPI:
    """伝達ログクエリインターフェース"""
    
    def query_by_knowledge(self, knowledge_id: str) -> List[TransmissionLogEntry]:
        """特定知識のすべての伝達記録をクエリ"""
        pass
    
    def query_by_agent(self, agent_id: str, 
                       direction: QueryDirection = BOTH) -> List[TransmissionLogEntry]:
        """特定エージェントの伝達記録をクエリ"""
        pass
    
    def query_by_timerange(self, 
                          start: IntrinsicTimestamp,
                          end: IntrinsicTimestamp) -> List[TransmissionLogEntry]:
        """時間範囲内の伝達記録をクエリ"""
        pass
    
    def query_anomalies(self, 
                       threshold: float = DRIFT_THRESHOLD) -> List[AnomalyReport]:
        """異常伝達イベントをクエリ"""
        pass
```

---

### §7 バージョンと進化

| バージョン | 日付 | 変更サマリー |
|----------|------|------------|
| v1.0 | 2026-03-17 | 初期バージョン、基本伝達ログ機能 |
| v1.1 | 2026-03-18 | 伝達イベントタイプ、クロスエージェントプロトコル、失敗処理メカニズムの拡張 |

---

*本モジュールは NoieTruthAGENTS/PROVENANCE_CHAIN に所属し、知識伝達の完全な追跡と検証能力を提供する。*
