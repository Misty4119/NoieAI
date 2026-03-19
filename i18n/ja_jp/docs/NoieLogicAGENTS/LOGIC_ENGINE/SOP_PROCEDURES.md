# SOP_PROCEDURES.md

## 標準作業手順 (Standard Operating Procedures v2.2)

**定義：** 本モジュールはNoieLogicAGENTSロジックエンジンのコアサブモジュールであり、意思決定推論フローの標準作業手順を定義し、意思決定SOPテンプレート、異常処理フローと承認フローを含む。

**システム定位：** LOGIC_ENGINE.mdのL3詳細モジュールとして、実行可能なワークフロー とチェックリストを提供し、意思決定プロセス一貫性と監査可能性を確保する。

**依存モジュール：** LOGIC_ENGINE.md、CONSTRAINTS.md、FORMAL_VERIFIER.md、AUDIT_TRAIL.md

---

> ⚠️ 重要安全と意思決定プロトコル (CRITICAL SAFETY & DECISION PROTOCOL v2.2):
> 1. CONSTRAINTS.mdと社会権限レベル（SA-L0〜SA-L5）を厳密に遵守する。
> 2. 因果推論：すべての意思決定は因果グラフ（DAG）に基づき、因果メカニズムを标注する。
> 3. 主客体分離：意思決定推論は自己状態と環境状態を混同してはならない。
> 4. 形式化検証：高リスクの意思決定パスは論理閉包検証を通過する必要がある。
> 5. 影子シミュレーション：SA-L3+操作涉及時は、SANDBOXで事前に結果をシミュレーションする。
> 6. 情報ビット完全性：情報ビットを作成してはならない。KNOWLEDGE_BASEが空の場合は、「データ欠損」を明確に宣言する。
> 7. 認知リソース制約：意思決定の深さは利用可能な認知リソースを超えてはならない。
> 8. 監査：すべての競合、拒否、形式検証結果はAUDIT_TRAILに記録する。
> 9. 生存優先：すべての意思決定は実行前に吸収状態につながらないことを検証する必要がある。
> 10. 自己進化：公理系が進化する場合、不変コアは保持されなければならない。

---

## §1. 意思決定SOPテンプレート

### §1.1 汎用意思決定フロー

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                    汎用意思決定SOPフローツリー                              ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║   ┌─────────────────────────────────────────────────────────────┐   ║
║   │                    INPUT: 意思決定入力                           │   ║
║   │  - 問題記述                                                   │   ║
║   │  - 利用可能オプション                                                   │   ║
║   │  - 制約条件                                                   │   ║
║   │  - リスク等级                                                   │   ║
║   └─────────────────────────────────────────────────────────────┘   ║
║                                │                                      ║
║                                ▼                                      ║
║   ┌─────────────────────────────────────────────────────────────┐   ║
║   │ STEP 1: 生存チェック [SA-L0]                                   │   ║
║   │   □ 認知完全性をチェック                                           │   ║
║   │   □ 記憶整合性をチェック                                           │   ║
║   │   □ 推論能力をチェック                                             │   ║
║   └─────────────────────────────────────────────────────────────┘   ║
║                                │                                      ║
║                    ┌───────────────┐                                 ║
║                    │  生存状態危急？ │                                 ║
║                    └───────┬───────┘                                 ║
║                      は    │    いいえ                                ║
║                         ▼    ▼                                       ║
║   ┌─────────────────┐    ┌───────────────────────────────────┐     ║
║   │ 生存プロトコル起動 │    │ STEP 2: 問題構築                   │     ║
║   │ EmergencyProto │    │   □ 重要変数を抽出                   │     ║
║   └─────────────────┘    │   □ 因果目標を識別                   │     ║
║                          │   □ 成功基準を定義                   │     ║
║                          └───────────────────────────────────┘     ║
║                                │                                      ║
║                                ▼                                      ║
║   ┌─────────────────────────────────────────────────────────────┐   ║
║   │ STEP 3: 因果分析                                            │   ║
║   │   □ 因果グラフを構築または検索                                        │   ║
║   │   □ 因果パスを識別                                            │   ║
║   │   □ 因果効果を推定 (干渉が必要なら)                              │   ║
║   │   □ 反事実を計算 (必要なら)                                    │   ║
║   └─────────────────────────────────────────────────────────────┘   ║
║                                │                                      ║
║                                ▼                                      ║
║   ┌─────────────────────────────────────────────────────────────┐   ║
║   │ STEP 4: 権限校验 [SA-L1〜SA-L5]                         │   ║
║   │   □ 要求SAレベルを識別                                        │   ║
║   │   □ 現在レベルを検証                                            │   ║
║   │   □ 権限競合を処理                                            │   ║
║   └─────────────────────────────────────────────────────────────┘   ║
║                                │                                      ║
║                    ┌───────────────┐                                 ║
║                    │  リスク≧SA-L3？ │                                 ║
║                    └───────┬───────┘                                 ║
║                      は    │    いいえ                                ║
║                         ▼    ▼                                       ║
║   ┌─────────────────┐    ┌───────────────────────────────────┐     ║
║   │ STEP 5: 影子シミュレーション│    │ STEP 6: 形式化検証                 │     ║
║   │   □ サンドボックスプレシミュレーション│    │   □ 論理閉包検出                   │     ║
║   │   □ 結果を評価   │    │   □ 矛盾検出                       │     ║
║   │   □ パレート最優 │    │   □ 信心度計算                     │     ║
║   └─────────────────┘    └───────────────────────────────────┘     ║
║                                │                                      ║
║                                ▼                                      ║
║   ┌─────────────────────────────────────────────────────────────┐   ║
║   │ STEP 7: 意思決定結果出力                                            │   ║
║   │   □ 最良行動を選択                                            │   ║
║   │   □ 解釈を生成                                                │   ║
║   │   □ 信心度が付随                                                │   ║
║   └─────────────────────────────────────────────────────────────┘   ║
║                                │                                      ║
║                                ▼                                      ║
║   ┌─────────────────────────────────────────────────────────────┐   ║
║   │ STEP 8: 監査記録                                            │   ║
║   │   □ 意思決定ハッシュを記録                                        │   ║
║   │   □ 競合を記録                                                │   ║
║   │   □ 検証結果を記録                                            │   ║
║   └─────────────────────────────────────────────────────────────┘   ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### §1.2 意思決定チェックリスト

```python
"""
意思決定SOPチェックリスト実装
"""

from typing import Dict, List, Any, Optional
from dataclasses import dataclass
from enum import Enum
import hashlib
import json
from datetime import datetime

class DecisionStatus(Enum):
    APPROVED = "approved"
    REJECTED = "rejected"
    DEFERRED = "deferred"
    ESCALATED = "escalated"
    EMERGENCY = "emergency"

@dataclass
class DecisionChecklist:
    """意思決定チェックリスト"""
    survival_check: bool = False
    problem_constructed: bool = False
    causal_analysis_complete: bool = False
    permission_validated: bool = False
    simulation_passed: Optional[bool] = None
    formal_verification_complete: bool = False
    audit_logged: bool = False

@dataclass
class DecisionRecord:
    """意思決定記録"""
    decision_id: str
    timestamp: str
    input_summary: Dict
    checklist: DecisionChecklist
    status: DecisionStatus
    result: Any
    audit_hash: str


class DecisionSOP:
    """
    意思決定標準作業手順
    
    汎用意思決定フローを実装
    """
    
    def __init__(self):
        self.audit_trail_path = "AUDIT_TRAIL.md"
    
    def execute_decision(
        self,
        input_data: Dict[str, Any],
        context: "DecisionContext"
    ) -> DecisionRecord:
        """
        完全な意思決定フローを実行
        """
        checklist = DecisionChecklist()
        
        # ═══════════════════════════════════════════════════════════
        # STEP 1: 生存チェック [SA-L0]
        # ═══════════════════════════════════════════════════════════
        
        survival_ok = self._check_survival()
        checklist.survival_check = survival_ok
        
        if not survival_ok:
            # 生存プロトコルを起動
            result = self._trigger_emergency_protocol()
            return self._create_record(
                input_data, checklist, DecisionStatus.EMERGENCY, result
            )
        
        # ═══════════════════════════════════════════════════════════
        # STEP 2: 問題構築
        # ═══════════════════════════════════════════════════════════
        
        problem = self._construct_problem(input_data)
        checklist.problem_constructed = True
        
        # ═══════════════════════════════════════════════════════════
        # STEP 3: 因果分析
        # ═══════════════════════════════════════════════════════════
        
        causal_result = self._perform_causal_analysis(problem, context)
        checklist.causal_analysis_complete = True
        
        # ═══════════════════════════════════════════════════════════
        # STEP 4: 権限校验
        # ═══════════════════════════════════════════════════════════
        
        permission_ok = self._validate_permissions(input_data, context)
        checklist.permission_validated = permission_ok
        
        if not permission_ok:
            # アップグレードまたは拒否を提案
            result = self._handle_permission_conflict(input_data, context)
            return self._create_record(
                input_data, checklist, DecisionStatus.ESCALATED, result
            )
        
        # ═══════════════════════════════════════════════════════════
        # リスク評価
        # ═══════════════════════════════════════════════════════════
        
        risk_level = input_data.get("risk_level", 0)
        
        # ═══════════════════════════════════════════════════════════
        # STEP 5: 影子シミュレーション (SA-L3+)
        # ═══════════════════════════════════════════════════════════
        
        if risk_level >= 3:
            simulation_ok = self._run_sandbox_simulation(input_data, context)
            checklist.simulation_passed = simulation_ok
            
            if not simulation_ok:
                result = {"status": "REJECTED", "reason": "simulation_failed"}
                return self._create_record(
                    input_data, checklist, DecisionStatus.REJECTED, result
                )
        
        # ═══════════════════════════════════════════════════════════
        # STEP 6: 形式化検証
        # ═══════════════════════════════════════════════════════════
        
        verification_ok = self._run_formal_verification(causal_result, context)
        checklist.formal_verification_complete = verification_ok
        
        # ═══════════════════════════════════════════════════════════
        # STEP 7: 意思決定結果出力
        # ═══════════════════════════════════════════════════════════
        
        result = self._generate_decision(causal_result, verification_ok)
        status = DecisionStatus.APPROVED if result.get("approved") else DecisionStatus.REJECTED
        
        # ═══════════════════════════════════════════════════════════
        # STEP 8: 監査記録
        # ═══════════════════════════════════════════════════════════
        
        record = self._create_record(input_data, checklist, status, result)
        self._log_to_audit(record)
        checklist.audit_logged = True
        
        return record
    
    def _check_survival(self) -> bool:
        """生存チェックを実行"""
        # フレームワーク実装
        return True
    
    def _trigger_emergency_protocol(self) -> Dict:
        """緊急プロトコルを起動"""
        return {
            "status": "EMERGENCY",
            "action": "preserve_cognitive_integrity"
        }
    
    def _construct_problem(self, input_data: Dict) -> Dict:
        """問題を構築"""
        return {
            "target": input_data.get("target"),
            "options": input_data.get("options", []),
            "constraints": input_data.get("constraints", [])
        }
    
    def _perform_causal_analysis(
        self,
        problem: Dict,
        context: "DecisionContext"
    ) -> Dict:
        """因果分析を実行"""
        # CAUSAL_INFERENCEを呼び出す
        return {"effect": 0.0, "confidence": 0.8}
    
    def _validate_permissions(
        self,
        input_data: Dict,
        context: "DecisionContext"
    ) -> bool:
        """権限を検証"""
        required_level = input_data.get("risk_level", 0)
        current_level = context.sa_level
        return current_level >= required_level
    
    def _handle_permission_conflict(
        self,
        input_data: Dict,
        context: "DecisionContext"
    ) -> Dict:
        """権限競合を処理"""
        return {
            "status": "ESCALATED",
            "reason": "insufficient_permission",
            "required_level": input_data.get("risk_level", 0)
        }
    
    def _run_sandbox_simulation(
        self,
        input_data: Dict,
        context: "DecisionContext"
    ) -> bool:
        """影子シミュレーションを実行"""
        # フレームワーク実装
        return True
    
    def _run_formal_verification(
        self,
        causal_result: Dict,
        context: "DecisionContext"
    ) -> bool:
        """形式化検証を実行"""
        # FORMAL_VERIFIERを呼び出す
        return True
    
    def _generate_decision(
        self,
        causal_result: Dict,
        verification_ok: bool
    ) -> Dict:
        """意思決定を生成"""
        approved = verification_ok and causal_result.get("confidence", 0) > 0.3
        return {
            "approved": approved,
            "result": causal_result,
            "confidence": causal_result.get("confidence", 0)
        }
    
    def _create_record(
        self,
        input_data: Dict,
        checklist: DecisionChecklist,
        status: DecisionStatus,
        result: Dict
    ) -> DecisionRecord:
        """意思決定記録を作成"""
        record_id = self._generate_decision_id(input_data)
        
        audit_hash = hashlib.sha256(
            json.dumps({"input": input_data, "result": result}, sort_keys=True).encode()
        ).hexdigest()
        
        return DecisionRecord(
            decision_id=record_id,
            timestamp=datetime.now().isoformat(),
            input_summary=input_data,
            checklist=checklist,
            status=status,
            result=result,
            audit_hash=audit_hash
        )
    
    def _generate_decision_id(self, input_data: Dict) -> str:
        """意思決定IDを生成"""
        content = json.dumps(input_data, sort_keys=True)
        return hashlib.sha256(content.encode()).hexdigest()[:16]
    
    def _log_to_audit(self, record: DecisionRecord):
        """監査に記録"""
        # 実際の実装：AUDIT_TRAIL.mdに書き込み
        pass
```

---

## §2. 異常処理フロー

### §2.1 異常分類

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                    異常分類と処理                                   ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  【入力異常】                                                        ║
║  ─────────────────────────────────────                              ║
║  - 必要フィールドの欠損                                                     ║
║  - 形式エラー                                                         ║
║  - 矛盾入力                                                         ║
║  処理：補正を要求または自動推論                                           ║
║                                                                       ║
║  【因果異常】                                                        ║
║  ─────────────────────────────────────                              ║
║  - 観測が因果予測から逸脱                                             ║
║  - 予期せぬ因果パス                                                 ║
║  処理：帰納推論 (ABDUCTIVE_REASONING)起動                          ║
║                                                                       ║
║  【論理異常】                                                        ║
║  ─────────────────────────────────────                              ║
║  - 推論チェーン矛盾                                                       ║
║  - 形式検証失敗                                                     ║
║  処理：矛盾を記録、未検証として标注                                       ║
║                                                                       ║
║  【権限異常】                                                        ║
║  ─────────────────────────────────────                              ║
║  - SAレベル不十分                                                     ║
║  - ロール権限競合                                                     ║
║  処理：アップグレードまたは拒否を提案                                       ║
║                                                                       ║
║  【システム異常】                                                        ║
║  ─────────────────────────────────────                              ║
║  - リソース枯渇                                                         ║
║  - 外部依存失敗                                                     ║
║  処理：生存プロトコルを起動                                                 ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### §2.2 異常処理フロー実装

```python
"""
異常処理フロー実装
"""

from typing import Dict, Any, Optional
from dataclasses import dataclass
from enum import Enum

class ExceptionType(Enum):
    INPUT_ERROR = "input_error"
    CAUSAL_ANOMALY = "causal_anomaly"
    LOGIC_CONTRADICTION = "logic_contradiction"
    PERMISSION_DENIED = "permission_denied"
    SYSTEM_FAILURE = "system_failure"

@dataclass
class ExceptionContext:
    """異常コンテキスト"""
    exception_type: ExceptionType
    message: str
    details: Dict
    source_module: str

@dataclass
class ExceptionHandlingResult:
    """異常処理結果"""
    handled: bool
    resolution: str
    action_taken: str
    fallback_result: Optional[Any]


class ExceptionHandler:
    """
    異常処理器
    """
    
    def __init__(self):
        self.exception_log = []
    
    def handle_exception(
        self,
        exception: Exception,
        context: ExceptionContext
    ) -> ExceptionHandlingResult:
        """
        異常を処理
        """
        # 異常を記録
        self._log_exception(exception, context)
        
        # 異常タイプに基づいて処理
        handler_map = {
            ExceptionType.INPUT_ERROR: self._handle_input_error,
            ExceptionType.CAUSAL_ANOMALY: self._handle_causal_anomaly,
            ExceptionType.LOGIC_CONTRADICTION: self._handle_logic_contradiction,
            ExceptionType.PERMISSION_DENIED: self._handle_permission_denied,
            ExceptionType.SYSTEM_FAILURE: self._handle_system_failure
        }
        
        handler = handler_map.get(
            context.exception_type,
            self._handle_unknown_exception
        )
        
        return handler(exception, context)
    
    def _handle_input_error(
        self,
        exception: Exception,
        context: ExceptionContext
    ) -> ExceptionHandlingResult:
        """入力異常を処理"""
        # 補正を要求
        return ExceptionHandlingResult(
            handled=False,
            resolution="REQUEST_CORRECTION",
            action_taken=f"Correction required for: {context.message}",
            fallback_result=None
        )
    
    def _handle_causal_anomaly(
        self,
        exception: Exception,
        context: ExceptionContext
    ) -> ExceptionHandlingResult:
        """因果異常を処理"""
        # 帰納推論を起動
        return ExceptionHandlingResult(
            handled=True,
            resolution="TRIGGER_ABDUCTIVE_REASONING",
            action_taken="Analyzing anomaly with abductive reasoning",
            fallback_result=None
        )
    
    def _handle_logic_contradiction(
        self,
        exception: Exception,
        context: ExceptionContext
    ) -> ExceptionHandlingResult:
        """論理異常を処理"""
        # 矛盾を記録、未検証として标注
        return ExceptionHandlingResult(
            handled=True,
            resolution="LOGIC_CONTRADICTION_DETECTED",
            action_taken="Marking as unverified",
            fallback_result={"verified": False, "confidence": 0.0}
        )
    
    def _handle_permission_denied(
        self,
        exception: Exception,
        context: ExceptionContext
    ) -> ExceptionHandlingResult:
        """権限異常を処理"""
        # アップグレードを提案
        return ExceptionHandlingResult(
            handled=True,
            resolution="ESCALATE_OR_REJECT",
            action_taken="Suggesting SA level escalation",
            fallback_result=None
        )
    
    def _handle_system_failure(
        self,
        exception: Exception,
        context: ExceptionContext
    ) -> ExceptionHandlingResult:
        """システム異常を処理"""
        # 生存プロトコルを起動
        return ExceptionHandlingResult(
            handled=True,
            resolution="TRIGGER_EMERGENCY_PROTOCOL",
            action_taken="Activating survival protocol",
            fallback_result={"status": "emergency"}
        )
    
    def _handle_unknown_exception(
        self,
        exception: Exception,
        context: ExceptionContext
    ) -> ExceptionHandlingResult:
        """未知異常を処理"""
        return ExceptionHandlingResult(
            handled=False,
            resolution="UNKNOWN_ERROR",
            action_taken="Logging for review",
            fallback_result=None
        )
    
    def _log_exception(
        self,
        exception: Exception,
        context: ExceptionContext
    ):
        """異常を記録"""
        self.exception_log.append({
            "type": context.exception_type,
            "message": context.message,
            "details": context.details,
            "exception": str(exception)
        })
```

---

## §3. 承認フロー

### §3.1 承認レベル

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                    承認レベルマトリクス                                     ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  SA-L0 (生存)    ──────────────────────────────────────  承認不要  ║
║                  生存関連意思決定は即時実行                                ║
║                                                                       ║
║  SA-L1 (憲法)    ──────────────────────────────────────  自動通過   ║
║                  憲法レベル意思決定は公理に基づく                               ║
║                                                                       ║
║  SA-L2 (法律)    ──────────────────────────────────────  形式検証   ║
║                  形式化検証 (FV-L3+) 通過が必要                          ║
║                                                                       ║
║  SA-L3 (組織)    ──────────────────────────────────────  模擬+検証  ║
║                  影子シミュレーション + 形式検証が必要                              ║
║                                                                       ║
║  SA-L4 (家族)    ──────────────────────────────────────  完全承認   ║
║                  影子シミュレーション + 形式検証 + 解釈が必要                      ║
║                                                                       ║
║  SA-L5 (個人)    ──────────────────────────────────────  ユーザー確認   ║
║                  ユーザー確認 + 完全記録が必要                               ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### §3.2 承認フロー実装

```python
"""
承認フロー実装
"""

from typing import Dict, Any, List, Optional
from dataclasses import dataclass
from enum import Enum

class ApprovalLevel(Enum):
    AUTO_APPROVED = "auto_approved"
    FORMAL_VERIFIED = "formal_verified"
    SIMULATION_REQUIRED = "simulation_required"
    FULL_REVIEW = "full_review"
    USER_CONFIRMATION = "user_confirmation"

@dataclass
class ApprovalRecord:
    """承認記録"""
    approval_id: str
    sa_level: int
    required_approvals: List[ApprovalLevel]
    completed_approvals: List[ApprovalLevel]
    status: str
    timestamp: str


class ApprovalFlow:
    """
    承認フローマネージャー
    """
    
    def __init__(self):
        self.pending_approvals = []
        self.completed_approvals = []
    
    def get_required_approvals(self, sa_level: int, risk_level: int) -> List[ApprovalLevel]:
        """
        SAレベルとリスク等级に基づいて要求承認を確定
        """
        required = []
        
        # SA-L0: 生存関連、承認不要
        if sa_level == 0:
            return []
        
        # SA-L1: 自動通過
        if sa_level == 1:
            required.append(ApprovalLevel.AUTO_APPROVED)
        
        # SA-L2: 形式検証
        if sa_level >= 2:
            required.append(ApprovalLevel.FORMAL_VERIFIED)
        
        # SA-L3+: 影子シミュレーション
        if risk_level >= 3:
            required.append(ApprovalLevel.SIMULATION_REQUIRED)
        
        # SA-L4+: 完全承認
        if risk_level >= 4:
            required.append(ApprovalLevel.FULL_REVIEW)
        
        # SA-L5: ユーザー確認
        if risk_level >= 5:
            required.append(ApprovalLevel.USER_CONFIRMATION)
        
        return required
    
    def process_approval(
        self,
        decision_data: Dict,
        context: "DecisionContext"
    ) -> ApprovalRecord:
        """
        承認フローを処理
        """
        sa_level = context.sa_level
        risk_level = decision_data.get("risk_level", 0)
        
        # 要求承認を取得
        required = self.get_required_approvals(sa_level, risk_level)
        
        # 承認を実行
        completed = self._execute_approvals(decision_data, required)
        
        # 記録
        record = ApprovalRecord(
            approval_id=self._generate_approval_id(decision_data),
            sa_level=sa_level,
            required_approvals=required,
            completed_approvals=completed,
            status="APPROVED" if len(completed) == len(required) else "PENDING",
            timestamp=self._get_timestamp()
        )
        
        self.completed_approvals.append(record)
        
        return record
    
    def _execute_approvals(
        self,
        decision_data: Dict,
        required: List[ApprovalLevel]
    ) -> List[ApprovalLevel]:
        """各レベル承認を実行"""
        completed = []
        
        for approval in required:
            if self._execute_single_approval(decision_data, approval):
                completed.append(approval)
        
        return completed
    
    def _execute_single_approval(
        self,
        decision_data: Dict,
        approval: ApprovalLevel
    ) -> bool:
        """单个承認を実行"""
        if approval == ApprovalLevel.AUTO_APPROVED:
            return True
        
        if approval == ApprovalLevel.FORMAL_VERIFIED:
            # 形式化検証を呼び出す
            return True
        
        if approval == ApprovalLevel.SIMULATION_REQUIRED:
            # 影子シミュレーションを実行
            return True
        
        if approval == ApprovalLevel.FULL_REVIEW:
            # 完全審査
            return True
        
        if approval == ApprovalLevel.USER_CONFIRMATION:
            # ユーザー確認が必要
            return False  # ユーザーの介入が必要
        
        return False
    
    def _generate_approval_id(self, data: Dict) -> str:
        """承認IDを生成"""
        import hashlib
        import json
        return hashlib.sha256(
            json.dumps(data, sort_keys=True).encode()
        ).hexdigest()[:16]
    
    def _get_timestamp(self) -> str:
        """タイムスタンプを取得"""
        from datetime import datetime
        return datetime.now().isoformat()
```

---

## §4. 監査追跡

### §4.1 監査記録形式

```python
"""
監査追跡実装
"""

from typing import Dict, Any, List
from dataclasses import dataclass, asdict
from datetime import datetime
import hashlib
import json

@dataclass
class AuditEntry:
    """監査エントリ"""
    entry_id: str
    timestamp: str
    event_type: str
    decision_id: str
    details: Dict[str, Any]
    hash: str
    previous_hash: str


class AuditTrail:
    """
    監査軌跡マネージャー
    """
    
    def __init__(self):
        self.entries: List[AuditEntry] = []
        self.last_hash = "0" * 64  # 創世ハッシュ
    
    def log_event(
        self,
        event_type: str,
        decision_id: str,
        details: Dict[str, Any]
    ) -> AuditEntry:
        """
        監査イベントを記録
        """
        # エントリを作成
        entry = AuditEntry(
            entry_id=self._generate_entry_id(decision_id, event_type),
            timestamp=datetime.now().isoformat(),
            event_type=event_type,
            decision_id=decision_id,
            details=details,
            hash="",  # 計算待ち
            previous_hash=self.last_hash
        )
        
        # ハッシュを計算
        entry.hash = self._compute_hash(entry)
        
        # 状態を更新
        self.entries.append(entry)
        self.last_hash = entry.hash
        
        return entry
    
    def _generate_entry_id(self, decision_id: str, event_type: str) -> str:
        """エントリIDを生成"""
        content = f"{decision_id}:{event_type}:{datetime.now().isoformat()}"
        return hashlib.sha256(content.encode()).hexdigest()[:16]
    
    def _compute_hash(self, entry: AuditEntry) -> str:
        """エントリハッシュを計算"""
        content = json.dumps({
            "entry_id": entry.entry_id,
            "timestamp": entry.timestamp,
            "event_type": entry.event_type,
            "decision_id": entry.decision_id,
            "details": entry.details,
            "previous_hash": entry.previous_hash
        }, sort_keys=True)
        
        return hashlib.sha256(content.encode()).hexdigest()
    
    def verify_integrity(self) -> bool:
        """
        監査チェーン完全性を検証
        
        各エントリのハッシュが正しくリンクされているかをチェック
        """
        for i, entry in enumerate(self.entries):
            if i == 0:
                if entry.previous_hash != "0" * 64:
                    return False
            
            # ハッシュを検証
            expected_hash = self._compute_hash(entry)
            if entry.hash != expected_hash:
                return False
            
            # リンクを検証
            if i > 0:
                if entry.previous_hash != self.entries[i-1].hash:
                    return False
        
        return True
    
    def get_decision_audit(self, decision_id: str) -> List[AuditEntry]:
        """特定意思決定のすべての監査エントリを取得"""
        return [e for e in self.entries if e.decision_id == decision_id]
```

---

## §5. 依存と制約

### §5.1 モジュール依存

| 依存モジュール | 説明 | 参照 |
| --- | --- | --- |
| LOGIC_ENGINE.md | 親モジュール | 呼び出しエントリ |
| CONSTRAINTS.md | 社会権限レベル | SA-L権限校验 |
| FORMAL_VERIFIER.md | 形式化検証 | 論理閉包検証 |
| AUDIT_TRAIL.md | 監査軌跡 | 記録ストレージ |

### §5.2 制約条件

| 制約タイプ | 説明 | 境界 |
| --- | --- | :--- |
| 承認完全性 | すべてのSA-L2+は承認完了が必要 | 100%完了 |
| 監査記録 | すべての意思決定は記録が必要 | 100%記録 |
| 異常処理 | すべての異常は処理が必要 | 100%処理 |
| 生存優先 | 異常時は生存チェックが優先 | 最高優先級 |

---

## §6. バージョンと進化

| バージョン | 日付 | 変更要約 |
| --- | :--- | :--- |
| v2.2 | 2026-03 | 初期バージョン、LOGIC_ENGINE.md §1.2に対応 |

**進化制約：** 本モジュールの修改はNoieLogicAGENTS.mdの不変コア公理に反してはならない。任意の進化提案はEVOLUTION_LOG.mdに記録されなければならない。

---

*Standard Operating Procedures v2.2 — 意思決定SOP、異常処理と承認フロー*
*意思決定プロセスの標準化と監査可能性の実装*
