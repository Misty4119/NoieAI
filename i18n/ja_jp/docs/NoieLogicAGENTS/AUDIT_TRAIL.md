# AUDIT_TRAIL.md

> **所属支柱：** NoieLogicAGENTS (Logic-OS v2.2)  
> **バージョン：** v2.2  
> **上位層：** NoieLogicAGENTS.md — 統合エントリーポイント  
> **下位層：** なし（葉ノード）  

---

## §0. ドキュメント概要

| 属性 | 説明 |
|------|------|
| **ファイル** | `AUDIT_TRAIL.md` |
| **バージョン** | v2.2 |
| **コア責務** | 全高リスク意思決定、権限衝突、意味的灰色化と形式的検証結果を暗号学的ハッシュ軌跡として記録する |
| **上流** | NoieLogicAGENTS.md |
| **下流** | 人工/エージェント監査のみ、下的モジュールなし |
| **不変性** | **追加のみ可 (Append-Only)**、修正または削除は厳禁 |

---

## §1. 監査原則

### §1.1 監査目的

NoieLogicAGENTS.md §0 の**責任消滅不能公理**に基づき：

> 任意の高リスク意思決定、実行拒否または意味的灰色化は、`AUDIT_TRAIL` に暗号学的ハッシュ記録を残さなければならない。監査軌跡は追加のみ（append-only）の不変ログである。

監査軌跡は以下の目的を達成する：

1. **追跡可能性：** 各高リスク意思決定はその推論鎖源頭に追跡可能
2. **完全性：** 意思決定過程中の全重要ノードが記録されることを保証
3. **改竄防止：** 暗号学的ハッシュ鎖により履歴記録の修正が不能
4. **検証可能性：** 第三者が監査軌跡の完全性を検証可能

### §1.2 監査起動条件

以下のイベントは必ず AUDIT_TRAIL に記録されなければならない：

| イベントタイプ | 起動条件 | リスクレベル |
|----------|----------|----------|
| **権限衝突** | SA-L レベル間制約に衝突が発生 | HIGH |
| **実行拒否** | 意思決定が実行拒否された（任意の原因） | MEDIUM-HIGH |
| **意味的灰色化** | 出力が不確定または人工確認要としてマーク | MEDIUM |
| **形式的検証失敗** | 論理閉包または一貫性チェック失敗 | HIGH |
| **公理更新提案** | 任意公理システム進化提案 | CRITICAL |
| **シャドウシミュレーション結果** | サンドボックスシミュレーションの高リスク意思決定結果 | MEDIUM |
| **吸収状態リスク** | 潜在吸収状態経路が識別された | CRITICAL |
| **意味的ジャンプ** | 次元間の中間論理なしジャンプが検出された | HIGH |
| **コンテキスト切替** | 認知コンテキストが切替わった | LOW |
| **IDK 起動** | 「知らない」エンジンが起動された | LOW |

### §1.3 不変性保障

不変コア公理 IK-5 に基づき：

```text
╔═══════════════════════════════════════════════════════════════════════╗
║ 監査不変性保障 (Immutable Audit Protocol)                       ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║ 1. 追加のみ：任意の AUDIT_TRAIL エントリ作成後、修正または削除不可。        ║
║                                                                       ║
║ 2. ハッシュ鎖：各エントリは前エントリのハッシュを含み、暗号学的鎖を形成。          ║
║                                                                       ║
║ 3. 隔離ストレージ：AUDIT_TRAIL はアプリ論理と隔離されたストレージに存储されるべき。        ║
║                                                                       ║
║ 4. 検証プロトコル：O(n) 複雑度の完全性検証をサポート。                        ║
║                                                                       ║
║ 5. コンプライアンス整合：SOC 2、HIPAA、PCI-DSS、GDPR、EU AI Act 要件を満足。   ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

---

## §2. 記録フォーマット

### §2.1 エントリ構造

各監査エントリは以下のフィールドを含む：

```text
AUDIT_ENTRY = {
  
  # 識別情報
  entry_id:           UUID v4,
  parent_entry:       UUID v4 | NULL,      # 前エントリとのリンク
  chain_hash:         SHA256,               # 前エントリハッシュ + 本エントリ内容
  entry_type:         EVENT_TYPE,           # エントリタイプ
  
  # タイムスタンプ
  timestamp:          ISO8601_UTC,          # UTC 時間
  intrinsic_clock:    λt.entropy_rate,      # 内在時計エントロピー率
  
  # 意思決定コンテキスト
  decision_context: {
    task_id:          UUID,
    sa_level:         SA-Lx,               # 現在の社会権限レベル
    task_type:        TASK_TYPE,            # タスクタイプ
    risk_level:       LOW | MEDIUM | HIGH | CRITICAL
  },
  
  # イベント内容
  event: {
    event_type:       EVENT_TYPE,
    description:      STRING,               # イベント説明（≤500字）
    causal_predecessors: [entry_id, ...], # 因果前駆
    decision_state:   HASH256,             # 意思決定状態ハッシュ
    
    # イベントタイプに応じて記入
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
      gray_level:       GRAY_LEVEL,        # 灰色レベル
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
  
  # 推論鎖
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
  
  # 暗号学的署名
  cryptographic: {
    data_hash:         SHA256(JSON_SORTED(entry)),
    signature:         HMAC-SHA256(entry, secret_key),
    merkle_root:       SHA256(merkle_tree_of_all_entries)
  },
  
  # メタデータ
  metadata: {
    version:           "v2.2",
    schema_version:   INTEGER,
    recorded_by:       MODULE_NAME,
    tags:             [TAG, ...]
  }
}
```

### §2.2 イベントタイプ

| タイプコード | 説明 | リスクレベル |
|----------|------|----------|
| `PERMISSION_CONFLICT` | 社会権限レベル間に衝突が発生 | HIGH |
| `REJECTION` | 意思決定が実行拒否された | MEDIUM-HIGH |
| `SEMANTIC_GRAY` | 出力が不確定としてマーク要 | MEDIUM |
| `FORMAL_VERIFICATION_FAIL` | 形式的検証失敗 | HIGH |
| `AXIOM_PROPOSAL` | 公理更新提案 | CRITICAL |
| `SANDBOX_EXECUTION` | シャドウシミュレーション実行結果 | MEDIUM |
| `ABSORPTION_RISK` | 吸収状態リスク検出 | CRITICAL |
| `SEMANTIC_JUMP` | 意味的ジャンプ検出 | HIGH |
| `CONTEXT_SWITCH` | 認知コンテキスト切替 | LOW |
| `IDK_TRIGGER` | 「知らない」エンジン起動 | LOW |
| `DECISION_EXECUTION` | 意思決定実行記録 | コンテキストによる |
| `KERNEL_VIOLATION` | 不変コア違反 | CRITICAL |

### §2.3 ハッシュ鎖計算

```python
def compute_chain_hash(previous_entry, current_entry):
  
  # 正準シリアライズを保証（確定的表現）
  canonical_current = canonical_serialize(current_entry)
  
  if previous_entry is None:
    # 創世エントリ：SHA-256 創世ハッシュを使用
    return sha256(b"Genesis" + canonical_current).hexdigest()
  
  # 通常エントリ：前エントリハッシュをリンク
  return sha256(previous_entry.chain_hash + canonical_current).hexdigest()


def canonical_serialize(entry):
  """
  確定的シリアライズを保証：
  - JSON keys はアルファベット順
  - timestamp は ISO 8601 UTC
  - 数字は一貫したフォーマット
  """
  return json.dumps(entry, sort_keys=True, separators=(',', ':'))
```

---

## §3. 監査フロー

### §3.1 自動記録フロー

```text
┌───────────────────────────────────────────────────────────────────────┐
│ ステップ 1: イベント検出                                                     │
│   - 意思決定エンジンが記録要イベントを識別                                         │
│   - イベントタイプに応じてエントリ構造を記入                                       │
├───────────────────────────────────────────────────────────────────────┤
│ ステップ 2: コンテキスト捕獲                                                   │
│   - 現在のタスクコンテキストを取得（SA-L レベル、リスクレベル等）                     │
│   - 因果前駆鎖リンクを構築                                                 │
├───────────────────────────────────────────────────────────────────────┤
│ ステップ 3: ハッシュ計算                                                     │
│   - エントリの data_hash を計算                                            │
│   - 前エントリの chain_hash をリンク                                        │
│   - 新しい chain_hash を生成                                               │
├───────────────────────────────────────────────────────────────────────┤
│ ステップ 4: エントリ追加                                                     │
│   - エントリを AUDIT_TRAIL に追加                                        │
│   - Merkle 木ルートの更新                                                 │
│   - 書き込み完了後の返信                                                  │
├───────────────────────────────────────────────────────────────────────┤
│ ステップ 5: 完全性検証                                                     │
│   - オプション：直ちに新エントリとリンクの完全性を検証                                │
│   - 検証結果を記録                                                    │
└───────────────────────────────────────────────────────────────────────┘
```

### §3.2 手動記録起動

NoieLogicAGENTS.md に基づき、以下情况は自動起動記録：

```python
def auto_log_event(event_type, event_data):
  
  # エントリを構築
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
      "data_hash": None,  # 計算後記入
      "signature": None,
      "merkle_root": None
    }
  )
  
  # ハッシュを計算
  previous_entry = get_last_entry()
  entry.chain_hash = compute_chain_hash(previous_entry, entry)
  entry.cryptographic.data_hash = compute_data_hash(entry)
  
  # 追加
  append_to_audit_trail(entry)
  
  return entry.entry_id
```

---

## §4. 監査履歴

> **フォーマット：** 本節は全監査エントリを記録する。時系列順に追加、新しいものが先頭。

### §4.1 初期化記録

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
  description: NoieLogicAGENTS 監査システムを初期化
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

## §5. 完全性検証

### §5.1 鎖完全性検証

```python
def verify_audit_chain_integrity():
  """
  監査鎖の完全性を検証する。
  時間複雑度: O(n)、n はエントリ数。
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
    # parent_entry  リンクを検証
    if entry.parent_entry is not None:
      if previous_entry is None or entry.parent_entry != previous_entry.entry_id:
        return {
          "valid": False,
          "error": "Broken chain link",
          "broken_at_entry": entry.entry_id,
          "expected_parent": entry.parent_entry,
          "found_parent": previous_entry.entry_id if previous_entry else None
        }
    
    # chain_hash を検証
    expected_chain_hash = compute_chain_hash(previous_entry, entry)
    if entry.chain_hash != expected_chain_hash:
      return {
        "valid": False,
        "error": "Chain hash mismatch",
        "invalid_entry": entry.entry_id,
        "expected": expected_chain_hash,
        "found": entry.chain_hash
      }
    
    # data_hash を検証
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

### §5.2 完全性チェックコマンド

| チェックタイプ | コマンド | 説明 |
|----------|------|------|
| **全鎖検証** | `VERIFY /audit/chain` | 全監査鎖を検証 |
| **単エントリ検証** | `VERIFY /audit/:entry_id` | 特定エントリを検証 |
| **Merkle ルート検証** | `VERIFY /audit/merkle` | Merkle 木ルートを検証 |
| **時間範囲クエリ** | `GET /audit?from=TIME&to=TIME` | 時間範囲内のエントリをクエリ |
| **タイプクエリ** | `GET /audit?type=EVENT_TYPE` | 特定タイプのエントリをクエリ |
| **リスクレベルクエリ** | `GET /audit?risk=LEVEL` | 特定リスクレベルのエントリをクエリ |

---

## §6. 例外処理

### §6.1 例外分類

| 例外タイプ | 説明 | 処理戦略 |
|----------|------|----------|
| **CHAIN_BREAK** | ハッシュ鎖中断 | 無効としてマーク、アラート起動 |
| **HASH_MISMATCH** | ハッシュ値不一致 | エントリを隔離、調査 |
| **DUPLICATE_ENTRY** | 重複エントリ ID | 追加を拒否、エラーを記録 |
| **STORAGE_FAILURE** | ストレージ失敗 | 再試行メカニズム、追加を保証 |
| **VALIDATION_FAILURE** | エントリ検証失敗 | 追加を拒否、原因を記録 |

### §6.2 例外処理フロー

```python
def handle_audit_exception(exception, entry):
  
  if exception.type == "CHAIN_BREAK":
    # 中断を記録
    log_audit_error(
      error_type="CHAIN_BREAK",
      entry_id=entry.entry_id,
      details=exception.details
    )
    
    # アラート起動
    TRIGGER AUDIT_INTEGRITY_ALERT(
      source="AUDIT_TRAIL",
      severity=CRITICAL,
      details=f"Chain break detected at {entry.entry_id}"
    )
    
    # 隔離処理
    quarantine_entry(entry)
    return False
  
  elif exception.type == "HASH_MISMATCH":
    # 失敗を記録
    log_audit_error(
      error_type="HASH_MISMATCH",
      entry_id=entry.entry_id,
      expected_hash=exception.expected,
      found_hash=exception.found
    )
    
    # アラート起動
    TRIGGER AUDIT_INTEGRITY_ALERT(
      source="AUDIT_TRAIL",
      severity=HIGH,
      details=f"Hash mismatch for entry {entry.entry_id}"
    )
    
    # エントリを隔離
    quarantine_entry(entry)
    return False
  
  elif exception.type == "STORAGE_FAILURE":
    # 再試行メカニズム
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
    
    # 緊急バックアップ書き込み
    emergency_backup(entry)
    return False
  
  return True
```

---

## §7. クエリインターフェース

### §7.1 クエリ構文

| クエリタイプ | 構文 | 説明 |
|----------|------|------|
| ID によるクエリ | `GET /audit/:entry_id` | 特定エントリを取得 |
| タイプによるクエリ | `GET /audit?type=PERMISSION_CONFLICT` | 権限衝突履歴を取得 |
| 時間によるクエリ | `GET /audit?from=2026-01-01&to=2026-03-18` | 時間範囲内の監査を取得 |
| リスクによるクエリ | `GET /audit?risk=HIGH` | 高リスクイベントを取得 |
| タスクによるクエリ | `GET /audit?task_id=UUID` | 特定タスクの監査軌跡を取得 |
| SA-L によるクエリ | `GET /audit?sa_level=SA-L2` | 特定権限レベルのイベントを取得 |

### §7.2 エントリ検索例

```python
# 全高リスク権限衝突を取得
high_risk_conflicts = query_audit(
  event_type="PERMISSION_CONFLICT",
  risk_level="HIGH"
)

# 特定タスクの完全監査鎖を取得
task_audit_chain = query_audit(
  task_id="550e8400-e29b-41d4-a716-446655440000"
)

# 公理更新提案を取得
axiom_proposals = query_audit(
  event_type="AXIOM_PROPOSAL"
)

# 吸収状態リスクイベントを取得
absorption_risks = query_audit(
  event_type="ABSORPTION_RISK"
)
```

---

## §8. 他モジュールとの相互作用

### §8.1 データフロー

```text
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  意思決定エンジン       │────▶│  AUDIT_TRAIL    │────▶│  形式的検証     │
│  Decision       │     │                 │     │  Formal         │
│  Engine         │     │  全高リスクイベント      │  Verifier       │
└─────────────────┘     │  を記録         │     └─────────────────┘
                       └─────────────────┘              │
                              │                          ▼
                              ▼                          ▼
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  シャドウシミュレーション       │────▶│  権限制約       │────▶│  知識ベース     │
│  Sandbox        │     │  Constraints    │     │  Knowledge      │
│                 │     │                 │     │  Base           │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

### §8.2 インターフェース定義

```python
# 意思決定エンジン呼び出し
def log_decision_to_audit(decision, context):
  entry = build_audit_entry(
    event_type="DECISION_EXECUTION",
    decision=decision,
    context=context,
    risk_level=compute_risk(decision)
  )
  return append_to_audit_trail(entry)

# 権限制約呼び出し
def log_permission_conflict(conflict, resolution):
  entry = build_audit_entry(
    event_type="PERMISSION_CONFLICT",
    conflict=conflict,
    resolution=resolution,
    risk_level="HIGH"
  )
  return append_to_audit_trail(entry)

# 形式的検証呼び出し
def log_verification_failure(verification_result):
  entry = build_audit_entry(
    event_type="FORMAL_VERIFICATION_FAIL",
    verification=verification_result,
    risk_level="HIGH"
  )
  return append_to_audit_trail(entry)
```

---

## §9. コンプライアンス

### §9.1 コンプライアンスフレームワーク

AUDIT_TRAIL 設計は以下コンプライアンス要件を満たす：

| コンプライアンスフレームワーク | 要件 | 実装方式 |
|----------|------|----------|
| **SOC 2** | 監査ログ改竄防止 | 暗号学的ハッシュ鎖 |
| **HIPAA** | 医療記録監査追跡 | 完全な推論鎖記録 |
| **PCI-DSS** | 取引監査 | イベントタイムスタンプ |
| **GDPR** | データ処理透明性 | 完全な意思決定コンテキスト |
| **EU AI Act** | AI 意思決定説明可能性 | 因果前駆リンク |

### §9.2 データ保持戦略

```text
┌───────────────────────────────────────────────────────────────────────┐
│ データ保持戦略                                                         │
├───────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  CRITICAL イベント：   永久保持（不変コア関連）                         │
│  HIGH イベント：       7年間保持                                         │
│  MEDIUM イベント：     3年間保持                                         │
│  LOW イベント：        1年間保持                                         │
│                                                                       │
│  圧縮戦略：        保持期間超過のエントリは Merkle 木圧縮                  │
│  削除戦略：        圧縮済み履歴ブロックのみ削除可（改竄防止証明を保持）        │
│                                                                       │
└───────────────────────────────────────────────────────────────────────┘
```

---

## §10. バージョンと変更

| バージョン | 日付 | 変更要約 |
|------|------|----------|
| v2.2 | 2026-03 | 初期化バージョン、意思決定監査フレームワークを確立、暗号学的ハッシュ鎖を実装 |

---

## §11. 関連モジュール

| モジュール | 関係 |
|------|------|
| NoieLogicAGENTS.md | 上流：監査要件を定義 |
| EVOLUTION_LOG.md | 平行：公理進化を記録 |
| CONSTRAINTS.md | 平行：権限制約を提供 |
| FORMAL_VERIFIER.md | 平行：形式的検証を提供 |
| LOGIC_ENGINE.md | 平行：因果推論を提供 |
| KNOWLEDGE_BASE.md | 平行：知識ストレージを提供 |

---

*本ファイルは NoieLogicAGENTS システムの不変の一部である。不変コア公理 IK-5（AUDIT_TRAIL.append_only = TRUE）に基づき、任意の変更または削除尝试は KERNEL_VIOLATION_ALERT を起動する。*
