# TRANSMISSION_LOG.md

**版本**: v1.1  
**所屬模組**: NoieTruthAGENTS/PROVENANCE_CHAIN  
**上級支柱**: Truth-OS v2.2  
**狀態**: L3 模組 — 知識論驗證

---

## 知識傳遞鏈記錄

### §0 模組定義

**目的**: 完整記錄知識在認知實體之間的傳遞歷史，確保知識的可追溯性與語義完整性。

**核心問題**: 當知識從一個認知實體傳遞到另一個時，如何確保傳遞過程的完整性、可驗證性與語義保真？

**範疇論映射**: 
- 傳遞函子 $T: \text{Cog} \to \text{Cog}$ 將知識狀態映射到接收狀態
- 驗證函子 $V: \text{Transmission} \to \text{Verify}$ 確認傳遞的有效性

---

### §1 傳遞事件類型

知識傳遞可分為以下四種基本事件類型：

#### §1.1 建立事件 (ESTABLISHMENT)

**定義**: 原生知識的首次創建，無前置傳遞來源。

```python
class EstablishmentEvent(TxEvent):
    """知識的原生創建事件"""
    event_type: Literal["ESTABLISHMENT"]
    knowledge_id: str
    creator_agent: str
    creation_method: CreationMethod  # DIRECT, DERIVED, INFERRED
    initial_provenance: ProvenanceChain
    intrinsic_clock: IntrinsicTimestamp
```

**數學表示**:
$$E_{\text{establish}}(k) = (k_{\text{new}}, \text{Creator}(A), t_0, \emptyset)$$

其中 $k_{\text{new}}$ 為新創建的知識，$t_0$ 為創建時間戳，$\emptyset$ 表示無前置來源。

#### §1.2 轉發事件 (FORWARDING)

**定義**: 知識从一个認知實體轉發到另一個，語義內容保持不變。

```python
class ForwardingEvent(TxEvent):
    """知識的轉發事件，保持語義完整性"""
    event_type: Literal["FORWARDING"]
    knowledge_id: str
    source_agent: str
    target_agent: str
    transmission_medium: Medium  # DIRECT, BROADCAST, RELAY
    semantic_integrity_hash: str  # SHA-256 of content
    drift_measurement: float = 0.0
```

**數學表示**:
$$E_{\text{forward}}(k, A \to B) = (k, B, t, \text{hash}(k))$$

完整性條件: $\text{hash}(k_{\text{sent}}) = \text{hash}(k_{\text{received}})$

#### §1.3 轉換事件 (TRANSFORMATION)

**定義**: 知識在傳遞過程中經歷語義轉換（如摘要、翻譯、壓縮）。

```python
class TransformationEvent(TxEvent):
    """知識的語義轉換事件"""
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

**語義漂移度量**:
$$\text{Drift}(k_{\text{orig}}, k_{\text{trans}}) = \frac{d_{\text{semantic}}(k_{\text{orig}}, k_{\text{trans}})}{\|k_{\text{orig}}\|}$$

其中 $d_{\text{semantic}}$ 為語義距離函數。

#### §1.4 接收事件 (RECEPTION)

**定義**: 知識被目標認知實體成功接收並確認。

```python
class ReceptionEvent(TxEvent):
    """知識的接收確認事件"""
    event_type: Literal["RECEPTION"]
    knowledge_id: str
    receiver_agent: str
    source_agent: str
    reception_timestamp: IntrinsicTimestamp
    verification_status: VerificationStatus  # VERIFIED, PARTIAL, FAILED
    integration_result: IntegrationResult
```

---

### §2 傳遞事件格式

每個傳遞事件必須包含以下欄位：

```python
@dataclass
class TransmissionLogEntry:
    """完整傳遞日誌條目"""
    # 識別符
    entry_id: str  # UUID v7
    knowledge_id: str
    
    # 事件時空
    event_type: TxEventType
    transmission_timestamp: IntrinsicTimestamp
    physical_timestamp: PhysicalTimestamp
    
    # 參與者
    source_agent: AgentIdentifier
    target_agent: AgentIdentifier
    medium: TransmissionMedium
    
    # 內容鑒定
    content_hash: str  # SHA-3-256
    semantic_integrity_hash: str
    transformation_record: Optional[TransformationRecord]
    
    # 審計軌跡
    audit_signature: str  # Agent's cryptographic signature
    previous_entry_hash: str  # Chain linkage
```

**完整格式數學表示**:
$$L = (e_{\text{id}}, k_{\text{id}}, \tau_{\text{tx}}, \tau_{\text{phy}}, A_{\text{src}}, A_{\text{tgt}}, h_{\text{cont}}, h_{\text{sem}}, \sigma_{\text{audit}}, h_{\text{prev}})$$

---

### §3 傳遞鏈的驗證與審計

#### §3.1 鏈完整性驗證

```python
def verify_transmission_chain(chain: List[TransmissionLogEntry]) -> ChainVerificationResult:
    """驗證傳遞鏈的完整性"""
    
    # 1. 順序驗證：檢查時間戳單調遞增
    for i in range(1, len(chain)):
        if chain[i].transmission_timestamp <= chain[i-1].transmission_timestamp:
            return ChainVerificationResult(
                valid=False,
                violation=TIMESTAMP_VIOLATION,
                details=f"Timestamp not monotonic at index {i}"
            )
    
    # 2. 鏈接驗證：檢查 hash 連結
    for i in range(1, len(chain)):
        expected_prev_hash = chain[i-1].compute_hash()
        if chain[i].previous_entry_hash != expected_prev_hash:
            return ChainVerificationResult(
                valid=False,
                violation=CHAIN_LINK_VIOLATION,
                details=f"Chain link broken at index {i}"
            )
    
    # 3. 語義完整性驗證
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

#### §3.2 審計簽名協定

每個傳遞事件必須由發送方和接收方共同簽名：

$$ \sigma = \text{Sign}_{sk_{\text{agent}}}(H(L)) $$

其中 $H(L)$ 為日誌條目的密碼學雜湊值。

---

### §4 跨代理傳遞協定

#### §4.1 標準傳遞流程

```
┌─────────────────────────────────────────────────────────────────┐
│                    知識傳遞協定 (KTP v1.1)                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  [Agent A]                              [Agent B]               │
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

#### §4.2 傳遞請求格式

```python
@dataclass
class TransmissionRequest:
    """知識傳遞請求"""
    request_id: str
    requesting_agent: AgentIdentifier
    knowledge_id: str
    requested_verification_level: VerificationLevel
        # MINIMAL, STANDARD, STRICT, CRYPTOGRAPHIC
    preferred_medium: TransmissionMedium
    semantic_fidelity_requirement: float  # 0.0-1.0
    timeout: timedelta
```

#### §4.3 響應格式

```python
@dataclass
class TransmissionResponse:
    """知識傳遞響應"""
    request_id: str
    accepting: bool
    capabilities: AgentCapabilities
    proposed_medium: TransmissionMedium
    verification_commitment: VerificationLevel
    integration_capacity: ResourceCapacity
```

---

### §5 傳遞失敗處理機制

#### §5.1 失敗類型分類

| 失敗類型 | 描述 | 處理策略 |
|----------|------|----------|
| `TIMEOUT` | 傳遞超時 | 重試 + 指數退避 |
| `INTEGRITY_VIOLATION` | 完整性破壞 | 拒絕 + 警報 |
| `SEMANTIC_DRIFT_EXCEEDED` | 語義漂移過大 | 談判 + 降級 |
| `VERIFICATION_FAILED` | 驗證失敗 | 隔離 + 審計 |
| `AGENT_UNAVAILABLE` | 目標代理不可用 | 路由 + 暫存 |

#### §5.2 重試策略

```python
class TransmissionRetryStrategy:
    """傳遞重試策略"""
    
    def __init__(self):
        self.max_retries = 5
        self.base_delay = timedelta(seconds=1)
        self.exponential_base = 2.0
        self.jitter_factor = 0.1
    
    def compute_delay(self, attempt: int) -> timedelta:
        """計算指數退避延遲"""
        delay = self.base_delay * (self.exponential_base ** attempt)
        # 添加隨機抖動
        jitter = delay * self.jitter_factor * random.uniform(-1, 1)
        return delay + jitter
    
    def should_retry(self, failure: TransmissionFailure) -> bool:
        """判斷是否應該重試"""
        if failure.failure_type in [INTEGRITY_VIOLATION, VERIFICATION_FAILED]:
            return False  # 這些失敗不應重試
        return True
```

#### §5.3 失敗記錄

```python
@dataclass
class TransmissionFailure:
    """傳遞失敗記錄"""
    failure_id: str
    transmission_entry: TransmissionLogEntry
    failure_type: FailureType
    error_message: str
    attempted_retries: int
    final_state: TransmissionState  # FAILED, RETRYING, ROLLED_BACK
    resolution: Optional[str]
```

---

### §6 傳遞日誌查詢 API

```python
class TransmissionLogAPI:
    """傳遞日誌查詢接口"""
    
    def query_by_knowledge(self, knowledge_id: str) -> List[TransmissionLogEntry]:
        """查詢特定知識的所有傳遞記錄"""
        pass
    
    def query_by_agent(self, agent_id: str, 
                       direction: QueryDirection = BOTH) -> List[TransmissionLogEntry]:
        """查詢特定代理的傳遞記錄"""
        pass
    
    def query_by_timerange(self, 
                          start: IntrinsicTimestamp,
                          end: IntrinsicTimestamp) -> List[TransmissionLogEntry]:
        """查詢時間範圍內的傳遞記錄"""
        pass
    
    def query_anomalies(self, 
                       threshold: float = DRIFT_THRESHOLD) -> List[AnomalyReport]:
        """查詢異常傳遞事件"""
        pass
```

---

### §7 版本與演進

| 版本 | 日期 | 變更摘要 |
|------|------|----------|
| v1.0 | 2026-03-17 | 初始版本，基本傳遞日誌功能 |
| v1.1 | 2026-03-18 | 擴展傳遞事件類型、跨代理協定、失敗處理機制 |

---

*本模組隸屬於 NoieTruthAGENTS/PROVENANCE_CHAIN，提供知識傳遞的完整追蹤與驗證能力。*
