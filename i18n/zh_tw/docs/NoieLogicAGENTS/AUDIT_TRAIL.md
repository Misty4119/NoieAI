# AUDIT_TRAIL.md

> **所屬支柱：** NoieLogicAGENTS (Logic-OS v2.2)  
> **版本：** v2.2  
> **上一層：** NoieLogicAGENTS.md — 統一入口  
> **下層：** 無（葉節點）  

---

## §0. 文件概述

| 屬性 | 描述 |
|------|------|
| **檔案** | `AUDIT_TRAIL.md` |
| **版本** | v2.2 |
| **核心職責** | 記錄所有高風險決策、權限衝突、語義灰化與形式驗證結果的密碼學雜湊軌跡 |
| **上游** | NoieLogicAGENTS.md |
| **下游** | 僅供人工/代理審閱，無下游模組 |
| **不可變性** | **僅可追加 (Append-Only)**，嚴禁任何修改或刪除 |

---

## §1. 審計原則

### §1.1 審計目標

根據 NoieLogicAGENTS.md §0 的**責任不可磨滅公理**：

> 任何高風險決策、拒絕執行或語義灰化，**必須**在 `AUDIT_TRAIL` 中留下密碼學雜湊紀錄。審計軌跡是僅可追加（append-only）的不可變日誌。

審計軌跡達成以下目標：

1. **可追溯性：** 每個高風險決策可追溯至其推論鏈源頭
2. **完整性：** 確保決策過程中所有關鍵節點被記錄
3. **不可篡改性：** 密碼學雜湊鏈確保任何歷史記錄不可被修改
4. **可驗證性：** 第三方可驗證審計軌跡的完整性

### §1.2 審計觸發條件

以下事件**必須**記錄至 AUDIT_TRAIL：

| 事件類型 | 觸發條件 | 風險等級 |
|----------|----------|----------|
| **權限衝突** | SA-L 層級間約束發生衝突 | HIGH |
| **拒絕執行** | 決策被拒絕執行（任何原因） | MEDIUM-HIGH |
| **語義灰化** | 輸出需標記為不確定或需人工確認 | MEDIUM |
| **形式驗證失敗** | 邏輯閉包或一致性檢查失敗 | HIGH |
| **公理更新提議** | 任何公理系統演化提議 | CRITICAL |
| **影子模擬結果** | 沙盒模擬的高風險決策結果 | MEDIUM |
| **吸收態風險** | 識別到潛在吸收態路徑 | CRITICAL |
| **語義跳躍** | 檢測到維度間無中間邏輯的跳躍 | HIGH |
| **上下文切換** | 認知上下文發生切換 | LOW |
| **IDK 觸發** | 「不知道」引擎被啟動 | LOW |

### §1.3 不可變性保障

根據不可變核心公理 IK-5：

```text
╔═══════════════════════════════════════════════════════════════════════╗
║ 審計不可變性保障 (Immutable Audit Protocol)                       ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║ 1. 僅可追加：任何 AUDIT_TRAIL 條目創建後，不可修改或刪除。        ║
║                                                                       ║
║ 2. 雜湊鏈：每個條目包含前一条目的雜湊，形成密碼學鏈接。          ║
║                                                                       ║
║ 3. 隔離存儲：AUDIT_TRAIL 應存儲於與應用邏輯隔離的存儲中。        ║
║                                                                       ║
║ 4. 驗證協議：支援 O(n) 複雜度的完整性驗證。                        ║
║                                                                       ║
║ 5. 合規對齊：滿足 SOC 2、HIPAA、PCI-DSS、GDPR、EU AI Act 要求。   ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

---

## §2. 記錄格式

### §2.1 條目結構

每個審計條目包含以下欄位：

```text
AUDIT_ENTRY = {
  
  # 識別資訊
  entry_id:           UUID v4,
  parent_entry:       UUID v4 | NULL,      # 鏈接前一条目
  chain_hash:         SHA256,               # 前一條目雜湊 + 本條目內容
  entry_type:         EVENT_TYPE,           # 條目類型
  
  # 時間戳記
  timestamp:          ISO8601_UTC,          # UTC 時間
  intrinsic_clock:    λt.entropy_rate,      # 內在時鐘熵率
  
  # 決策上下文
  decision_context: {
    task_id:          UUID,
    sa_level:         SA-Lx,               # 當前社會權限層級
    task_type:        TASK_TYPE,            # 任務類型
    risk_level:       LOW | MEDIUM | HIGH | CRITICAL
  },
  
  # 事件內容
  event: {
    event_type:       EVENT_TYPE,
    description:      STRING,               # 事件描述（≤500字）
    causal_predecessors: [entry_id, ...], # 因果前驅
    decision_state:   HASH256,             # 決策狀態雜湊
    
    # 根據事件類型填寫
    permission_conflict: {
      conflicting_levels: [SA-Lx, SA-Ly],
      resolution:         STRING,
      overriding_level:   SA-Lx
    } | null,
    
    rejection_details: {
      reason:           STRING,
      alternative:      STRING | null,
      blocked_by:       CONSTRAINT_ID
    } | null,
    
    semantic_gray: {
      gray_level:       GRAY_LEVEL,        # 灰度等級
      confidence:       0.0-1.0,
      requires_human:   BOOLEAN
    } | null,
    
    formal_verification: {
      status:           PASS | FAIL,
      proof_id:         UUID | null,
      verification_type: CLOSURE | CONSISTENCY | PROVABILITY,
      issues:           [ISSUE, ...] | null
    } | null,
    
    axiom_proposal: {
      proposal_id:      UUID,
      target_module:    MODULE_NAME,
      change_summary:   STRING,
      kernel_affected:  [IK-x, ...]
    } | null,
    
    sandbox_result: {
      simulation_id:    UUID,
      outcome:         SAFE | UNSAFE | BOUNDARY,
      metrics:         { metric: value, ... }
    } | null,
    
    absorption_risk: {
      risk_detected:    BOOLEAN,
      absorption_path:  [STATE, ...],
      mitigation:       STRING
    } | null,
    
    semantic_jump: {
      from_dimension:    DIMENSION,
      to_dimension:     DIMENSION,
      intermediate_logic: [STEP, ...],
      violation:        BOOLEAN
    } | null,
    
    idk_trigger: {
      ignorance_type:   KK | KU | UK | UU | Π | UD,
      exploration_gradient: VECTOR
    } | null
  },
  
  # 推論鏈
  inference_chain: {
    root_cause:        AXIOM_ID | FACT_ID,
    chain:            [ {
      step_id:         UUID,
      premise:         STRING,
      inference_rule:  STRING,
      conclusion:      STRING
    }, ... ],
    depth:             INTEGER
  },
  
  # 密碼學簽名
  cryptographic: {
    data_hash:         SHA256(JSON_SORTED(entry)),
    signature:         HMAC-SHA256(entry, secret_key),
    merkle_root:       SHA256(merkle_tree_of_all_entries)
  },
  
  # 元數據
  metadata: {
    version:           "v2.2",
    schema_version:   INTEGER,
    recorded_by:       MODULE_NAME,
    tags:             [TAG, ...]
  }
}
```

### §2.2 事件類型

| 類型代碼 | 描述 | 風險等級 |
|----------|------|----------|
| `PERMISSION_CONFLICT` | 社會權限層級間發生衝突 | HIGH |
| `REJECTION` | 決策被拒絕執行 | MEDIUM-HIGH |
| `SEMANTIC_GRAY` | 輸出需標記為不確定 | MEDIUM |
| `FORMAL_VERIFICATION_FAIL` | 形式化驗證失敗 | HIGH |
| `AXIOM_PROPOSAL` | 公理更新提議 | CRITICAL |
| `SANDBOX_EXECUTION` | 影子模擬執行結果 | MEDIUM |
| `ABSORPTION_RISK` | 吸收態風險偵測 | CRITICAL |
| `SEMANTIC_JUMP` | 語義跳躍偵測 | HIGH |
| `CONTEXT_SWITCH` | 認知上下文切換 | LOW |
| `IDK_TRIGGER` | 「不知道」引擎觸發 | LOW |
| `DECISION_EXECUTION` | 決策執行記錄 | 按情境 |
| `KERNEL_VIOLATION` | 不可變核心違規 | CRITICAL |

### §2.3 雜湊鏈計算

```python
def compute_chain_hash(previous_entry, current_entry):
  
  # 確保 canonical 序列化（確定性表示）
  canonical_current = canonical_serialize(current_entry)
  
  if previous_entry is None:
    # 創世條目：使用 SHA-256 創世雜湊
    return sha256(b"Genesis" + canonical_current).hexdigest()
  
  # 普通條目：鏈接前一条目雜湊
  return sha256(previous_entry.chain_hash + canonical_current).hexdigest()


def canonical_serialize(entry):
  """
  確保確定性序列化：
  - JSON keys 按字母排序
  - timestamp 使用 ISO 8601 UTC
  - 數字使用一致性格式
  """
  return json.dumps(entry, sort_keys=True, separators=(',', ':'))
```

---

## §3. 審計流程

### §3.1 自動記錄流程

```text
┌───────────────────────────────────────────────────────────────────────┐
│ 步驟 1: 事件偵測                                                     │
│   - 決策引擎識別需記錄的事件                                         │
│   - 根據事件類型填充條目結構                                         │
├───────────────────────────────────────────────────────────────────────┤
│ 步驟 2: 上下文捕獲                                                   │
│   - 獲取當前任務上下文（SA-L 等級、風險等級等）                     │
│   - 建構因果前驅鏈接                                                 │
├───────────────────────────────────────────────────────────────────────┤
│ 步驟 3: 雜湊計算                                                     │
│   - 計算條目的 data_hash                                            │
│   - 鏈接前一条目的 chain_hash                                        │
│   - 生成新的 chain_hash                                             │
├───────────────────────────────────────────────────────────────────────┤
│ 步驟 4: 條目追加                                                     │
│   - 將條目追加到 AUDIT_TRAIL                                        │
│   - 更新 Merkle 樹根                                                 │
│   - 確保寫入完成後返回                                              │
├───────────────────────────────────────────────────────────────────────┤
│ 步驟 5: 完整性驗證                                                   │
│   - 可選：立即驗證新條目與鏈接完整性                                │
│   - 記錄驗證結果                                                    │
└───────────────────────────────────────────────────────────────────────┘
```

### §3.2 手動記錄觸發

根據 NoieLogicAGENTS.md，以下情況自動觸發記錄：

```python
def auto_log_event(event_type, event_data):
  
  # 構建條目
  entry = AUDIT_ENTRY(
    entry_id=uuid4(),
    parent_entry=get_last_entry_id(),
    entry_type=event_type,
    timestamp=utcnow_iso8601(),
    intrinsic_clock=compute_intrinsic_clock(),
    decision_context=get_current_context(),
    event=event_data,
    inference_chain=reconstruct_inference_chain(event_data),
    cryptographic={
      "data_hash": None,  # 計算後填充
      "signature": None,
      "merkle_root": None
    }
  )
  
  # 計算雜湊
  previous_entry = get_last_entry()
  entry.chain_hash = compute_chain_hash(previous_entry, entry)
  entry.cryptographic.data_hash = compute_data_hash(entry)
  
  # 追加
  append_to_audit_trail(entry)
  
  return entry.entry_id
```

---

## §4. 審計歷史

> **格式：** 本節記錄所有審計條目。按時間順序追加，newest at top。

### §4.1 初始化記錄

```text
================================================================================
AUDIT_ENTRY id: audit-init-0000-0000-0000-0000
parent_entry: null
chain_hash: e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
entry_type: SYSTEM_INIT
timestamp: 2026-03-18T00:00:00Z
intrinsic_clock: λt.0

decision_context:
  task_id: null
  sa_level: SA-L0
  task_type: INITIALIZATION
  risk_level: CRITICAL

event:
  event_type: SYSTEM_INIT
  description: 初始化 NoieLogicAGENTS 審計系統
  causal_predecessors: []
  decision_state: 5d41402abc4b2a76b9719d911017c592
  
  permission_conflict: null
  rejection_details: null
  semantic_gray: null
  formal_verification: null
  axiom_proposal: null
  sandbox_result: null
  absorption_risk: null
  semantic_jump: null
  idk_trigger: null

inference_chain:
  root_cause: SYSTEM_INIT
  chain: []
  depth: 0

cryptographic:
  data_hash: 5d41402abc4b2a76b9719d911017c592
  signature: 8f4e8c3d9a2b5f6e1c7d8a9b0c1e2f3
  merkle_root: a1b2c3d4e5f678901234567890123456789

metadata:
  version: v2.2
  schema_version: 1
  recorded_by: SYSTEM_INIT
  tags: [initialization, audit_system_start]
================================================================================
```

---

## §5. 完整性驗證

### §5.1 鏈完整性驗證

```python
def verify_audit_chain_integrity():
  """
  驗證審計鏈的完整性。
  時間複雜度: O(n)，其中 n 為條目數量。
  """
  
  entries = load_all_entries("AUDIT_TRAIL")
  
  if len(entries) == 0:
    return {
      "valid": True,
      "entry_count": 0,
      "message": "Empty audit trail"
    }
  
  previous_entry = None
  
  for i, entry in enumerate(entries):
    # 驗證 parent_entry 鏈接
    if entry.parent_entry is not None:
      if previous_entry is None or entry.parent_entry != previous_entry.entry_id:
        return {
          "valid": False,
          "error": "Broken chain link",
          "broken_at_entry": entry.entry_id,
          "expected_parent": entry.parent_entry,
          "found_parent": previous_entry.entry_id if previous_entry else None
        }
    
    # 驗證 chain_hash
    expected_chain_hash = compute_chain_hash(previous_entry, entry)
    if entry.chain_hash != expected_chain_hash:
      return {
        "valid": False,
        "error": "Chain hash mismatch",
        "invalid_entry": entry.entry_id,
        "expected": expected_chain_hash,
        "found": entry.chain_hash
      }
    
    # 驗證 data_hash
    expected_data_hash = compute_data_hash(entry)
    if entry.cryptographic.data_hash != expected_data_hash:
      return {
        "valid": False,
        "error": "Data hash mismatch",
        "invalid_entry": entry.entry_id
      }
    
    previous_entry = entry
  
  return {
    "valid": True,
    "entry_count": len(entries),
    "first_entry": entries[0].entry_id,
    "last_entry": entries[-1].entry_id,
    "last_chain_hash": entries[-1].chain_hash
  }
```

### §5.2 完整性檢查命令

| 檢查類型 | 命令 | 描述 |
|----------|------|------|
| **完整鏈驗證** | `VERIFY /audit/chain` | 驗證整條審計鏈 |
| **單條目驗證** | `VERIFY /audit/:entry_id` | 驗證特定條目 |
| **Merkle 根驗證** | `VERIFY /audit/merkle` | 驗證 Merkle 樹根 |
| **時間範圍查詢** | `GET /audit?from=TIME&to=TIME` | 查詢時間範圍內的條目 |
| **類型查詢** | `GET /audit?type=EVENT_TYPE` | 查詢特定類型的條目 |
| **風險等級查詢** | `GET /audit?risk=LEVEL` | 查詢特定風險等級的條目 |

---

## §6. 異常處理

### §6.1 異常分類

| 異常類型 | 描述 | 處理策略 |
|----------|------|----------|
| **CHAIN_BREAK** | 雜湊鏈中斷 | 標記為無效，觸發警報 |
| **HASH_MISMATCH** | 雜湊值不匹配 | 隔離條目，進行調查 |
| **DUPLICATE_ENTRY** | 重複條目 ID | 拒絕追加，記錄錯誤 |
| **STORAGE_FAILURE** | 存儲失敗 | 重試機制，確保追加成功 |
| **VALIDATION_FAILURE** | 條目驗證失敗 | 拒絕追加，記錄原因 |

### §6.2 異常處理流程

```python
def handle_audit_exception(exception, entry):
  
  if exception.type == "CHAIN_BREAK":
    # 記錄中斷
    log_audit_error(
      error_type="CHAIN_BREAK",
      entry_id=entry.entry_id,
      details=exception.details
    )
    
    # 觸發警報
    TRIGGER AUDIT_INTEGRITY_ALERT(
      source="AUDIT_TRAIL",
      severity=CRITICAL,
      details=f"Chain break detected at {entry.entry_id}"
    )
    
    # 隔離處理
    quarantine_entry(entry)
    return False
  
  elif exception.type == "HASH_MISMATCH":
    # 記錄失敗
    log_audit_error(
      error_type="HASH_MISMATCH",
      entry_id=entry.entry_id,
      expected_hash=exception.expected,
      found_hash=exception.found
    )
    
    # 觸發警報
    TRIGGER AUDIT_INTEGRITY_ALERT(
      source="AUDIT_TRAIL",
      severity=HIGH,
      details=f"Hash mismatch for entry {entry.entry_id}"
    )
    
    # 隔離條目
    quarantine_entry(entry)
    return False
  
  elif exception.type == "STORAGE_FAILURE":
    # 重試機制
    for attempt in range(MAX_RETRY_ATTEMPTS):
      try:
        append_to_audit_trail(entry)
        return True
      except StorageException:
        wait(RETRY_DELAY * (attempt + 1))
    
    # 最終失敗
    TRIGGER AUDIT_SYSTEM_FAILURE(
      source="AUDIT_TRAIL",
      severity=CRITICAL,
      details="Failed to append entry after max retries"
    )
    
    # 緊急寫入備份
    emergency_backup(entry)
    return False
  
  return True
```

---

## §7. 查詢介面

### §7.1 查詢語法

| 查詢類型 | 語法 | 描述 |
|----------|------|------|
| 按 ID 查詢 | `GET /audit/:entry_id` | 獲取特定條目 |
| 按類型查詢 | `GET /audit?type=PERMISSION_CONFLICT` | 獲取權限衝突歷史 |
| 按時間查詢 | `GET /audit?from=2026-01-01&to=2026-03-18` | 獲取時間範圍內的審計 |
| 按風險查詢 | `GET /audit?risk=HIGH` | 獲取高風險事件 |
| 按任務查詢 | `GET /audit?task_id=UUID` | 獲取特定任務的審計軌跡 |
| 按 SA-L 查詢 | `GET /audit?sa_level=SA-L2` | 獲取特定權限層級的事件 |

### §7.2 條目檢索示例

```python
# 獲取所有高風險權限衝突
high_risk_conflicts = query_audit(
  event_type="PERMISSION_CONFLICT",
  risk_level="HIGH"
)

# 獲取特定任務的完整審計鏈
task_audit_chain = query_audit(
  task_id="550e8400-e29b-41d4-a716-446655440000"
)

# 獲取公理更新提議
axiom_proposals = query_audit(
  event_type="AXIOM_PROPOSAL"
)

# 獲取吸收態風險事件
absorption_risks = query_audit(
  event_type="ABSORPTION_RISK"
)
```

---

## §8. 與其他模組的交互

### §8.1 數據流

```text
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  決策引擎       │────▶│  AUDIT_TRAIL    │────▶│  形式化驗證     │
│  Decision       │     │                 │     │  Formal         │
│  Engine         │     │  記錄所有       │     │  Verifier       │
└─────────────────┘     │  高風險事件      │     └─────────────────┘
                       └─────────────────┘              │
                              │                          │
                              ▼                          ▼
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  影子模擬       │────▶│  權限約束       │────▶│  知識庫         │
│  Sandbox        │     │  Constraints    │     │  Knowledge      │
│                 │     │                 │     │  Base           │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

### §8.2 接口定義

```python
# 決策引擎調用
def log_decision_to_audit(decision, context):
  entry = build_audit_entry(
    event_type="DECISION_EXECUTION",
    decision=decision,
    context=context,
    risk_level=compute_risk(decision)
  )
  return append_to_audit_trail(entry)

# 權限約束調用
def log_permission_conflict(conflict, resolution):
  entry = build_audit_entry(
    event_type="PERMISSION_CONFLICT",
    conflict=conflict,
    resolution=resolution,
    risk_level="HIGH"
  )
  return append_to_audit_trail(entry)

# 形式化驗證調用
def log_verification_failure(verification_result):
  entry = build_audit_entry(
    event_type="FORMAL_VERIFICATION_FAIL",
    verification=verification_result,
    risk_level="HIGH"
  )
  return append_to_audit_trail(entry)
```

---

## §9. 合規性

### §9.1 合規框架

AUDIT_TRAIL 設計滿足以下合規要求：

| 合規框架 | 要求 | 實現方式 |
|----------|------|----------|
| **SOC 2** | 審計日誌不可篡改 | 密碼學雜湊鏈 |
| **HIPAA** | 醫療記錄審計追蹤 | 完整推論鏈記錄 |
| **PCI-DSS** | 交易審計 | 事件時間戳記 |
| **GDPR** | 數據處理透明性 | 完整決策上下文 |
| **EU AI Act** | AI 決策可解釋性 | 因果前驅鏈接 |

### §9.2 數據保留策略

```text
┌───────────────────────────────────────────────────────────────────────┐
│ 數據保留策略                                                         │
├───────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  CRITICAL 事件：   永久保留（不可變核心相關）                         │
│  HIGH 事件：       保留 7 年                                          │
│  MEDIUM 事件：     保留 3 年                                          │
│  LOW 事件：        保留 1 年                                          │
│                                                                       │
│  壓縮策略：        超過保留期的條目進行 Merkle 樹壓縮                  │
│  刪除策略：        僅可刪除已壓縮的歷史區塊（不可篡改證明保留）        │
│                                                                       │
└───────────────────────────────────────────────────────────────────────┘
```

---

## §10. 版本與變更

| 版本 | 日期 | 變更摘要 |
|------|------|----------|
| v2.2 | 2026-03 | 初始化版本，建立決策審計框架，實現密碼學雜湊鏈 |

---

## §11. 相關模組

| 模組 | 關係 |
|------|------|
| NoieLogicAGENTS.md | 上游：定義審計要求 |
| EVOLUTION_LOG.md | 平行：記錄公理演化 |
| CONSTRAINTS.md | 平行：提供權限約束 |
| FORMAL_VERIFIER.md | 平行：提供形式化驗證 |
| LOGIC_ENGINE.md | 平行：提供因果推論 |
| KNOWLEDGE_BASE.md | 平行：提供知識存儲 |

---

*本文件是 NoieLogicAGENTS 系統不可變的一部分。根據不可變核心公理 IK-5（AUDIT_TRAIL.append_only = TRUE），任何修改或刪除嘗試將觸發 KERNEL_VIOLATION_ALERT。*
