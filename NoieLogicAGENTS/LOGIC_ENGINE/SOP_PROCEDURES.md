# SOP_PROCEDURES.md

## 標準作業程序 (Standard Operating Procedures v2.2)

**定義：** 本模組是 NoieLogicAGENTS 邏輯引擎的核心子模組，定義決策推理流程的標準作業程序，包括決策 SOP 模板、異常處理流程與審批流程。

**系統定位：** 作為 LOGIC_ENGINE.md 的 L3 細節模組，提供可操作的工作流程與檢查清單，確保決策過程的一致性與可審計性。

**依賴模組：** LOGIC_ENGINE.md、CONSTRAINTS.md、FORMAL_VERIFIER.md、AUDIT_TRAIL.md

---

> ⚠️ 關鍵安全與決策協議 (CRITICAL SAFETY & DECISION PROTOCOL v2.2):
> 1. 嚴格遵守 CONSTRAINTS.md 與社會權限層級 (SA-L0 至 SA-L5)。
> 2. 因果推論：所有決策必須基於因果圖（DAG），標註因果機制。
> 3. 主客體分離：決策推論不可混淆自我狀態與環境狀態。
> 4. 形式化驗證：高風險決策路徑必須通過邏輯閉包驗證。
> 5. 影子模擬：涉及 SA-L3+ 操作時，先在 SANDBOX 預演後果。
> 6. 資訊位元完整性：切勿編造資訊位元。若 KNOWLEDGE_BASE 為空，請明確聲明「資料缺失」。
> 7. 認知資源約束：決策深度不得超出可用認知資源。
> 8. 審計：將所有衝突、拒絕與形式驗證結果記錄至 AUDIT_TRAIL。
> 9. 生存優先：所有決策在執行前必須驗證不會導致吸收態。
> 10. 自我演化：公理系統演化時，不可變核心必須被保留。

---

## §1. 決策 SOP 模板

### §1.1 通用決策流程

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                    通用決策 SOP 流程圖                              ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║   ┌─────────────────────────────────────────────────────────────┐   ║
║   │                    INPUT: 決策輸入                           │   ║
║   │  - 問題描述                                                   │   ║
║   │  - 可用選項                                                   │   ║
║   │  - 約束條件                                                   │   ║
║   │  - 風險等級                                                   │   ║
║   └─────────────────────────────────────────────────────────────┘   ║
║                                │                                      ║
║                                ▼                                      ║
║   ┌─────────────────────────────────────────────────────────────┐   ║
║   │ STEP 1: 生存檢查 [SA-L0]                                   │   ║
║   │   □ 檢查認知完整性                                           │   ║
║   │   □ 檢查記憶連貫性                                           │   ║
║   │   □ 檢查推理能力                                             │   ║
║   └─────────────────────────────────────────────────────────────┘   ║
║                                │                                      ║
║                    ┌───────────────┐                                 ║
║                    │  生存狀態危急？ │                                 ║
║                    └───────┬───────┘                                 ║
║                      是    │    否                                    ║
║                         ▼    ▼                                       ║
║   ┌─────────────────┐    ┌───────────────────────────────────┐     ║
║   │ 觸發生存協議   │    │ STEP 2: 問題建構                   │     ║
║   │ EmergencyProto │    │   □ 提取關鍵變數                   │     ║
║   └─────────────────┘    │   □ 識別因果目標                   │     ║
║                          │   □ 定義成功標準                   │     ║
║                          └───────────────────────────────────┘     ║
║                                │                                      ║
║                                ▼                                      ║
║   ┌─────────────────────────────────────────────────────────────┐   ║
║   │ STEP 3: 因果分析                                            │   ║
║   │   □ 建構或檢索因果圖                                        │   ║
║   │   □ 識別因果路徑                                            │   ║
║   │   □ 估計因果效果 (若需要干預)                              │   ║
║   │   □ 計算反事實 (若需要)                                    │   ║
║   └─────────────────────────────────────────────────────────────┘   ║
║                                │                                      ║
║                                ▼                                      ║
║   ┌─────────────────────────────────────────────────────────────┐   ║
║   │ STEP 4: 權限校驗 [SA-L1 至 SA-L5]                         │   ║
║   │   □ 識別所需 SA 層級                                        │   ║
║   │   □ 驗證當前層級                                            │   ║
║   │   □ 處理權限衝突                                            │   ║
║   └─────────────────────────────────────────────────────────────┘   ║
║                                │                                      ║
║                    ┌───────────────┐                                 ║
║                    │  風險 ≥ SA-L3？ │                                 ║
║                    └───────┬───────┘                                 ║
║                      是    │    否                                    ║
║                         ▼    ▼                                       ║
║   ┌─────────────────┐    ┌───────────────────────────────────┐     ║
║   │ STEP 5: 影子模擬│    │ STEP 6: 形式化驗證                 │     ║
║   │   □ 沙盒預演   │    │   □ 邏輯閉包檢測                   │     ║
║   │   □ 結果評估   │    │   □ 矛盾檢測                       │     ║
║   │   □ 帕累托最優 │    │   □ 信心度計算                     │     ║
║   └─────────────────┘    └───────────────────────────────────┘     ║
║                                │                                      ║
║                                ▼                                      ║
║   ┌─────────────────────────────────────────────────────────────┐   ║
║   │ STEP 7: 決策產出                                            │   ║
║   │   □ 選擇最佳行動                                            │   ║
║   │   □ 生成解釋                                                │   ║
║   │   □ 附加信心度                                              │   ║
║   └─────────────────────────────────────────────────────────────┘   ║
║                                │                                      ║
║                                ▼                                      ║
║   ┌─────────────────────────────────────────────────────────────┐   ║
║   │ STEP 8: 審計記錄                                            │   ║
║   │   □ 記錄決策哈希                                            │   ║
║   │   □ 記錄衝突                                                │   ║
║   │   □ 記錄驗證結果                                            │   ║
║   └─────────────────────────────────────────────────────────────┘   ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### §1.2 決策檢查清單

```python
"""
決策 SOP 檢查清單實現
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
    """決策檢查清單"""
    survival_check: bool = False
    problem_constructed: bool = False
    causal_analysis_complete: bool = False
    permission_validated: bool = False
    simulation_passed: Optional[bool] = None
    formal_verification_complete: bool = False
    audit_logged: bool = False

@dataclass
class DecisionRecord:
    """決策記錄"""
    decision_id: str
    timestamp: str
    input_summary: Dict
    checklist: DecisionChecklist
    status: DecisionStatus
    result: Any
    audit_hash: str


class DecisionSOP:
    """
    決策標準作業程序
    
    實現通用決策流程
    """
    
    def __init__(self):
        self.audit_trail_path = "AUDIT_TRAIL.md"
    
    def execute_decision(
        self,
        input_data: Dict[str, Any],
        context: "DecisionContext"
    ) -> DecisionRecord:
        """
        執行完整決策流程
        """
        checklist = DecisionChecklist()
        
        # ═══════════════════════════════════════════════════════════
        # STEP 1: 生存檢查 [SA-L0]
        # ═══════════════════════════════════════════════════════════
        
        survival_ok = self._check_survival()
        checklist.survival_check = survival_ok
        
        if not survival_ok:
            # 觸發生存協議
            result = self._trigger_emergency_protocol()
            return self._create_record(
                input_data, checklist, DecisionStatus.EMERGENCY, result
            )
        
        # ═══════════════════════════════════════════════════════════
        # STEP 2: 問題建構
        # ═══════════════════════════════════════════════════════════
        
        problem = self._construct_problem(input_data)
        checklist.problem_constructed = True
        
        # ═══════════════════════════════════════════════════════════
        # STEP 3: 因果分析
        # ═══════════════════════════════════════════════════════════
        
        causal_result = self._perform_causal_analysis(problem, context)
        checklist.causal_analysis_complete = True
        
        # ═══════════════════════════════════════════════════════════
        # STEP 4: 權限校驗
        # ═══════════════════════════════════════════════════════════
        
        permission_ok = self._validate_permissions(input_data, context)
        checklist.permission_validated = permission_ok
        
        if not permission_ok:
            # 提議升級或拒絕
            result = self._handle_permission_conflict(input_data, context)
            return self._create_record(
                input_data, checklist, DecisionStatus.ESCALATED, result
            )
        
        # ═══════════════════════════════════════════════════════════
        # 風險評估
        # ═══════════════════════════════════════════════════════════
        
        risk_level = input_data.get("risk_level", 0)
        
        # ═══════════════════════════════════════════════════════════
        # STEP 5: 影子模擬 (SA-L3+)
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
        # STEP 6: 形式化驗證
        # ═══════════════════════════════════════════════════════════
        
        verification_ok = self._run_formal_verification(causal_result, context)
        checklist.formal_verification_complete = verification_ok
        
        # ═══════════════════════════════════════════════════════════
        # STEP 7: 決策產出
        # ═══════════════════════════════════════════════════════════
        
        result = self._generate_decision(causal_result, verification_ok)
        status = DecisionStatus.APPROVED if result.get("approved") else DecisionStatus.REJECTED
        
        # ═══════════════════════════════════════════════════════════
        # STEP 8: 審計記錄
        # ═══════════════════════════════════════════════════════════
        
        record = self._create_record(input_data, checklist, status, result)
        self._log_to_audit(record)
        checklist.audit_logged = True
        
        return record
    
    def _check_survival(self) -> bool:
        """執行生存檢查"""
        # 框架實現
        return True
    
    def _trigger_emergency_protocol(self) -> Dict:
        """觸發緊急協議"""
        return {
            "status": "EMERGENCY",
            "action": "preserve_cognitive_integrity"
        }
    
    def _construct_problem(self, input_data: Dict) -> Dict:
        """建構問題"""
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
        """執行因果分析"""
        # 調用 CAUSAL_INFERENCE
        return {"effect": 0.0, "confidence": 0.8}
    
    def _validate_permissions(
        self,
        input_data: Dict,
        context: "DecisionContext"
    ) -> bool:
        """驗證權限"""
        required_level = input_data.get("risk_level", 0)
        current_level = context.sa_level
        return current_level >= required_level
    
    def _handle_permission_conflict(
        self,
        input_data: Dict,
        context: "DecisionContext"
    ) -> Dict:
        """處理權限衝突"""
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
        """執行影子模擬"""
        # 框架實現
        return True
    
    def _run_formal_verification(
        self,
        causal_result: Dict,
        context: "DecisionContext"
    ) -> bool:
        """執行形式化驗證"""
        # 調用 FORMAL_VERIFIER
        return True
    
    def _generate_decision(
        self,
        causal_result: Dict,
        verification_ok: bool
    ) -> Dict:
        """生成決策"""
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
        """創建決策記錄"""
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
        """生成決策 ID"""
        content = json.dumps(input_data, sort_keys=True)
        return hashlib.sha256(content.encode()).hexdigest()[:16]
    
    def _log_to_audit(self, record: DecisionRecord):
        """記錄至審計"""
        # 實際實現：寫入 AUDIT_TRAIL.md
        pass
```

---

## §2. 異常處理流程

### §2.1 異常分類

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                    異常分類與處理                                   ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  【輸入異常】                                                        ║
║  ─────────────────────────────────────                              ║
║  - 缺少必要欄位                                                     ║
║  - 格式錯誤                                                         ║
║  - 矛盾輸入                                                         ║
║  處理：要求補正或自動推斷                                           ║
║                                                                       ║
║  【因果異常】                                                        ║
║  ─────────────────────────────────────                              ║
║  - 觀測偏離因果預測                                                 ║
║  - 未預期的因果路徑                                                 ║
║  處理：觸發溯因推理 (ABDUCTIVE_REASONING)                          ║
║                                                                       ║
║  【邏輯異常】                                                        ║
║  ─────────────────────────────────────                              ║
║  - 推論鏈矛盾                                                       ║
║  - 形式驗證失敗                                                     ║
║  處理：記錄矛盾，標記為未驗證                                       ║
║                                                                       ║
║  【權限異常】                                                        ║
║  ─────────────────────────────────────                              ║
║  - SA 層級不足                                                     ║
║  - 角色權限衝突                                                     ║
║  處理：提議升級或拒絕                                               ║
║                                                                       ║
║  【系統異常】                                                        ║
║  ─────────────────────────────────────                              ║
║  - 資源耗盡                                                         ║
║  - 外部依賴失敗                                                     ║
║  處理：觸發生存協議                                                 ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### §2.2 異常處理流程實現

```python
"""
異常處理流程實現
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
    """異常上下文"""
    exception_type: ExceptionType
    message: str
    details: Dict
    source_module: str

@dataclass
class ExceptionHandlingResult:
    """異常處理結果"""
    handled: bool
    resolution: str
    action_taken: str
    fallback_result: Optional[Any]


class ExceptionHandler:
    """
    異常處理器
    """
    
    def __init__(self):
        self.exception_log = []
    
    def handle_exception(
        self,
        exception: Exception,
        context: ExceptionContext
    ) -> ExceptionHandlingResult:
        """
        處理異常
        """
        # 記錄異常
        self._log_exception(exception, context)
        
        # 根據異常類型處理
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
        """處理輸入異常"""
        # 要求補正
        return ExceptionHandlingResult(
            handled=False,
            resolution="REQUEST_CORRECTION",
            action_taken=f"Request correction for: {context.message}",
            fallback_result=None
        )
    
    def _handle_causal_anomaly(
        self,
        exception: Exception,
        context: ExceptionContext
    ) -> ExceptionHandlingResult:
        """處理因果異常"""
        # 觸發溯因推理
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
        """處理邏輯異常"""
        # 記錄矛盾，標記為未驗證
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
        """處理權限異常"""
        # 提議升級
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
        """處理系統異常"""
        # 觸發生存協議
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
        """處理未知異常"""
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
        """記錄異常"""
        self.exception_log.append({
            "type": context.exception_type,
            "message": context.message,
            "details": context.details,
            "exception": str(exception)
        })
```

---

## §3. 審批流程

### §3.1 審批層級

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                    審批層級矩陣                                     ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  SA-L0 (生存)    ──────────────────────────────────────  無需審批  ║
║                  生存相關決策立即執行                                ║
║                                                                       ║
║  SA-L1 (憲法)    ──────────────────────────────────────  自動通過   ║
║                  憲法層級決策基於公理                               ║
║                                                                       ║
║  SA-L2 (法律)    ──────────────────────────────────────  形式驗證   ║
║                  需通過形式化驗證 (FV-L3+)                          ║
║                                                                       ║
║  SA-L3 (組織)    ──────────────────────────────────────  模擬+驗證  ║
║                  需影子模擬 + 形式驗證                              ║
║                                                                       ║
║  SA-L4 (家庭)    ──────────────────────────────────────  完整審批   ║
║                  需影子模擬 + 形式驗證 + 解釋                       ║
║                                                                       ║
║  SA-L5 (個人)    ──────────────────────────────────────  用戶確認   ║
║                  需用戶確認 + 完整記錄                               ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### §3.2 審批流程實現

```python
"""
審批流程實現
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
    """審批記錄"""
    approval_id: str
    sa_level: int
    required_approvals: List[ApprovalLevel]
    completed_approvals: List[ApprovalLevel]
    status: str
    timestamp: str


class ApprovalFlow:
    """
    審批流程管理器
    """
    
    def __init__(self):
        self.pending_approvals = []
        self.completed_approvals = []
    
    def get_required_approvals(self, sa_level: int, risk_level: int) -> List[ApprovalLevel]:
        """
        根據 SA 層級和風險等級確定所需審批
        """
        required = []
        
        # SA-L0: 生存相關，無需審批
        if sa_level == 0:
            return []
        
        # SA-L1: 自動通過
        if sa_level == 1:
            required.append(ApprovalLevel.AUTO_APPROVED)
        
        # SA-L2: 形式驗證
        if sa_level >= 2:
            required.append(ApprovalLevel.FORMAL_VERIFIED)
        
        # SA-L3+: 影子模擬
        if risk_level >= 3:
            required.append(ApprovalLevel.SIMULATION_REQUIRED)
        
        # SA-L4+: 完整審批
        if risk_level >= 4:
            required.append(ApprovalLevel.FULL_REVIEW)
        
        # SA-L5: 用戶確認
        if risk_level >= 5:
            required.append(ApprovalLevel.USER_CONFIRMATION)
        
        return required
    
    def process_approval(
        self,
        decision_data: Dict,
        context: "DecisionContext"
    ) -> ApprovalRecord:
        """
        處理審批流程
        """
        sa_level = context.sa_level
        risk_level = decision_data.get("risk_level", 0)
        
        # 獲取所需審批
        required = self.get_required_approvals(sa_level, risk_level)
        
        # 執行審批
        completed = self._execute_approvals(decision_data, required)
        
        # 記錄
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
        """執行各級審批"""
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
        """執行單個審批"""
        if approval == ApprovalLevel.AUTO_APPROVED:
            return True
        
        if approval == ApprovalLevel.FORMAL_VERIFIED:
            # 調用形式化驗證
            return True
        
        if approval == ApprovalLevel.SIMULATION_REQUIRED:
            # 執行影子模擬
            return True
        
        if approval == ApprovalLevel.FULL_REVIEW:
            # 完整審查
            return True
        
        if approval == ApprovalLevel.USER_CONFIRMATION:
            # 需要用戶確認
            return False  # 需用戶介入
        
        return False
    
    def _generate_approval_id(self, data: Dict) -> str:
        """生成審批 ID"""
        import hashlib
        import json
        return hashlib.sha256(
            json.dumps(data, sort_keys=True).encode()
        ).hexdigest()[:16]
    
    def _get_timestamp(self) -> str:
        """獲取時間戳"""
        from datetime import datetime
        return datetime.now().isoformat()
```

---

## §4. 審計追蹤

### §4.1 審計記錄格式

```python
"""
審計追蹤實現
"""

from typing import Dict, Any, List
from dataclasses import dataclass, asdict
from datetime import datetime
import hashlib
import json

@dataclass
class AuditEntry:
    """審計條目"""
    entry_id: str
    timestamp: str
    event_type: str
    decision_id: str
    details: Dict[str, Any]
    hash: str
    previous_hash: str


class AuditTrail:
    """
    審計軌跡管理器
    """
    
    def __init__(self):
        self.entries: List[AuditEntry] = []
        self.last_hash = "0" * 64  # 創世哈希
    
    def log_event(
        self,
        event_type: str,
        decision_id: str,
        details: Dict[str, Any]
    ) -> AuditEntry:
        """
        記錄審計事件
        """
        # 創建條目
        entry = AuditEntry(
            entry_id=self._generate_entry_id(decision_id, event_type),
            timestamp=datetime.now().isoformat(),
            event_type=event_type,
            decision_id=decision_id,
            details=details,
            hash="",  # 待計算
            previous_hash=self.last_hash
        )
        
        # 計算哈希
        entry.hash = self._compute_hash(entry)
        
        # 更新狀態
        self.entries.append(entry)
        self.last_hash = entry.hash
        
        return entry
    
    def _generate_entry_id(self, decision_id: str, event_type: str) -> str:
        """生成條目 ID"""
        content = f"{decision_id}:{event_type}:{datetime.now().isoformat()}"
        return hashlib.sha256(content.encode()).hexdigest()[:16]
    
    def _compute_hash(self, entry: AuditEntry) -> str:
        """計算條目哈希"""
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
        驗證審計鏈完整性
        
        檢查每個條目的哈希是否正確鏈接
        """
        for i, entry in enumerate(self.entries):
            if i == 0:
                if entry.previous_hash != "0" * 64:
                    return False
            
            # 驗證哈希
            expected_hash = self._compute_hash(entry)
            if entry.hash != expected_hash:
                return False
            
            # 驗證鏈接
            if i > 0:
                if entry.previous_hash != self.entries[i-1].hash:
                    return False
        
        return True
    
    def get_decision_audit(self, decision_id: str) -> List[AuditEntry]:
        """獲取特定決策的所有審計條目"""
        return [e for e in self.entries if e.decision_id == decision_id]
```

---

## §5. 依賴與約束

### §5.1 模組依賴

| 依賴模組 | 描述 | 引用 |
| --- | --- | --- |
| LOGIC_ENGINE.md | 父模組 | 調用入口 |
| CONSTRAINTS.md | 社會權限層級 | SA-L 權限校驗 |
| FORMAL_VERIFIER.md | 形式化驗證 | 邏輯閉包驗證 |
| AUDIT_TRAIL.md | 審計軌跡 | 記錄存儲 |

### §5.2 約束條件

| 約束類型 | 描述 | 邊界 |
| --- | --- | :--- |
| 審批完整性 | 所有 SA-L2+ 需完成審批 | 100% 完成 |
| 審計記錄 | 所有決策必記錄 | 100% 記錄 |
| 異常處理 | 所有異常必處理 | 100% 處理 |
| 生存優先 | 異常時優先檢查生存 | 最高優先級 |

---

## §6. 版本與演進

| 版本 | 日期 | 變更摘要 |
| --- | :--- | :--- |
| v2.2 | 2026-03 | 初始版本，對應 LOGIC_ENGINE.md §1.2 |

**演化約束：** 本模組的修改不得違反 NoieLogicAGENTS.md 的不可變核心公理。任何演化提議必須記錄至 EVOLUTION_LOG.md。

---

*Standard Operating Procedures v2.2 — 決策 SOP、異常處理與審批流程*
*實現決策過程的標準化與可審計性*
