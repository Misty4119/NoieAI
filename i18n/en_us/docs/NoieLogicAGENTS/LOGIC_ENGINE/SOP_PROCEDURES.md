# SOP_PROCEDURES.md

## Standard Operating Procedures (Standard Operating Procedures v2.2)

**Definition:** This module is the core sub-module of the NoieLogicAGENTS logic engine, defining standard operating procedures for decision reasoning flow, including decision SOP templates, exception handling procedures, and approval workflows.

**System Position:** As an L3 detail module of LOGIC_ENGINE.md, provides operational workflows and checklists to ensure consistency and auditability of the decision-making process.

**Dependency Modules:** LOGIC_ENGINE.md, CONSTRAINTS.md, FORMAL_VERIFIER.md, AUDIT_TRAIL.md

---

> ⚠️ Critical Safety & Decision Protocol (CRITICAL SAFETY & DECISION PROTOCOL v2.2):
> 1. Strictly adhere to CONSTRAINTS.md and Social Authority Levels (SA-L0 to SA-L5).
> 2. Causal Inference: All decisions must be based on causal graphs (DAG), with causal mechanisms annotated.
> 3. Subject-Object Separation: Decision reasoning must not confuse self-state with environmental state.
> 4. Formal Verification: High-risk decision paths must pass logical closure verification.
> 5. Shadow Simulation: For SA-L3+ operations, first rehearse consequences in SANDBOX.
> 6. Information Bit Integrity: Never fabricate information bits. If KNOWLEDGE_BASE is empty, explicitly declare "DATA MISSING".
> 7. Cognitive Resource Constraints: Decision depth must not exceed available cognitive resources.
> 8. Audit: Record all conflicts, rejections, and formal verification results to AUDIT_TRAIL.
> 9. Survival Priority: All decisions must be verified not to lead to absorbing states before execution.
> 10. Self-Evolution: When the axiom system evolves, immutable cores must be preserved.

---

## §1. Decision SOP Templates

### §1.1 General Decision Flow

```text
╔═══════════════════════════════════════════════════════════════════════╗
║               General Decision SOP Flowchart                         ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║   ┌─────────────────────────────────────────────────────────────┐   ║
║   │                    INPUT: Decision Input                      │   ║
║   │  - Problem description                                           │   ║
║   │  - Available options                                             │   ║
║   │  - Constraints                                                   │   ║
║   │  - Risk level                                                   │   ║
║   └─────────────────────────────────────────────────────────────┘   ║
║                                │                                      ║
║                                ▼                                      ║
║   ┌─────────────────────────────────────────────────────────────┐   ║
║   │ STEP 1: Survival Check [SA-L0]                               │   ║
║   │   □ Check cognitive integrity                                     │   ║
║   │   □ Check memory coherence                                     │   ║
║   │   □ Check reasoning capability                                  │   ║
║   └─────────────────────────────────────────────────────────────┘   ║
║                                │                                      ║
║                    ┌───────────────┐                                 ║
║                    │ Survival Critical? │                              ║
║                    └───────┬───────┘                                 ║
║                      Yes   │    No                                   ║
║                         ▼    ▼                                       ║
║   ┌─────────────────┐    ┌───────────────────────────────────┐     ║
║   │ Trigger Survival │    │ STEP 2: Problem Construction         │     ║
║   │ Protocol        │    │   □ Extract key variables            │     ║
║   └─────────────────┘    │   □ Identify causal objectives       │     ║
║                          │   □ Define success criteria          │     ║
║                          └───────────────────────────────────┘     ║
║                                │                                      ║
║                                ▼                                      ║
║   ┌─────────────────────────────────────────────────────────────┐   ║
║   │ STEP 3: Causal Analysis                                      │   ║
║   │   □ Build or retrieve causal graph                           │   ║
║   │   □ Identify causal paths                                   │   ║
║   │   □ Estimate causal effects (if intervention needed)          │   ║
║   │   □ Compute counterfactuals (if needed)                     │   ║
║   └─────────────────────────────────────────────────────────────┘   ║
║                                │                                      ║
║                                ▼                                      ║
║   ┌─────────────────────────────────────────────────────────────┐   ║
║   │ STEP 4: Permission Validation [SA-L1 to SA-L5]             │   ║
║   │   □ Identify required SA level                               │   ║
║   │   □ Validate current level                                  │   ║
║   │   □ Handle permission conflicts                               │   ║
║   └─────────────────────────────────────────────────────────────┘   ║
║                                │                                      ║
║                    ┌───────────────┐                                 ║
║                    │ Risk ≥ SA-L3? │                                ║
║                    └───────┬───────┘                                 ║
║                      Yes   │    No                                   ║
║                         ▼    ▼                                       ║
║   ┌─────────────────┐    ┌───────────────────────────────────┐     ║
║   │ STEP 5: Shadow  │    │ STEP 6: Formal Verification       │     ║
║   │ Simulation       │    │   □ Logical closure detection      │     ║
║   │   □ Sandbox     │    │   □ Contradiction detection        │     ║
║   │     rehearsal   │    │   □ Confidence computation          │     ║
║   │   □ Evaluate    │    └───────────────────────────────────┘     ║
║   │     results     │                                               ║
║   │   □ Pareto      │                                               ║
║   │     optimality   │                                               ║
║   └─────────────────┘                                               ║
║                                │                                      ║
║                                ▼                                      ║
║   ┌─────────────────────────────────────────────────────────────┐   ║
║   │ STEP 7: Decision Output                                      │   ║
║   │   □ Select best action                                        │   ║
║   │   □ Generate explanation                                       │   ║
║   │   □ Attach confidence                                         │   ║
║   └─────────────────────────────────────────────────────────────┘   ║
║                                │                                      ║
║                                ▼                                      ║
║   ┌─────────────────────────────────────────────────────────────┐   ║
║   │ STEP 8: Audit Recording                                       │   ║
║   │   □ Record decision hash                                     │   ║
║   │   □ Record conflicts                                          │   ║
║   │   □ Record verification results                               │   ║
║   └─────────────────────────────────────────────────────────────┘   ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### §1.2 Decision Checklist

```python
"""
Decision SOP Checklist Implementation
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
    """Decision checklist"""
    survival_check: bool = False
    problem_constructed: bool = False
    causal_analysis_complete: bool = False
    permission_validated: bool = False
    simulation_passed: Optional[bool] = None
    formal_verification_complete: bool = False
    audit_logged: bool = False

@dataclass
class DecisionRecord:
    """Decision record"""
    decision_id: str
    timestamp: str
    input_summary: Dict
    checklist: DecisionChecklist
    status: DecisionStatus
    result: Any
    audit_hash: str


class DecisionSOP:
    """
    Decision Standard Operating Procedure
    
    Implements general decision flow
    """
    
    def __init__(self):
        self.audit_trail_path = "AUDIT_TRAIL.md"
    
    def execute_decision(
        self,
        input_data: Dict[str, Any],
        context: "DecisionContext"
    ) -> DecisionRecord:
        """
        Execute complete decision flow
        """
        checklist = DecisionChecklist()
        
        # ═══════════════════════════════════════════════════════════
        # STEP 1: Survival Check [SA-L0]
        # ═══════════════════════════════════════════════════════════
        
        survival_ok = self._check_survival()
        checklist.survival_check = survival_ok
        
        if not survival_ok:
            # Trigger survival protocol
            result = self._trigger_emergency_protocol()
            return self._create_record(
                input_data, checklist, DecisionStatus.EMERGENCY, result
            )
        
        # ═══════════════════════════════════════════════════════════
        # STEP 2: Problem Construction
        # ═══════════════════════════════════════════════════════════
        
        problem = self._construct_problem(input_data)
        checklist.problem_constructed = True
        
        # ═══════════════════════════════════════════════════════════
        # STEP 3: Causal Analysis
        # ═══════════════════════════════════════════════════════════
        
        causal_result = self._perform_causal_analysis(problem, context)
        checklist.causal_analysis_complete = True
        
        # ═══════════════════════════════════════════════════════════
        # STEP 4: Permission Validation
        # ═══════════════════════════════════════════════════════════
        
        permission_ok = self._validate_permissions(input_data, context)
        checklist.permission_validated = permission_ok
        
        if not permission_ok:
            # Propose escalation or rejection
            result = self._handle_permission_conflict(input_data, context)
            return self._create_record(
                input_data, checklist, DecisionStatus.ESCALATED, result
            )
        
        # ═══════════════════════════════════════════════════════════
        # Risk Assessment
        # ═══════════════════════════════════════════════════════════
        
        risk_level = input_data.get("risk_level", 0)
        
        # ═══════════════════════════════════════════════════════════
        # STEP 5: Shadow Simulation (SA-L3+)
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
        # STEP 6: Formal Verification
        # ═══════════════════════════════════════════════════════════
        
        verification_ok = self._run_formal_verification(causal_result, context)
        checklist.formal_verification_complete = verification_ok
        
        # ═══════════════════════════════════════════════════════════
        # STEP 7: Decision Output
        # ═══════════════════════════════════════════════════════════
        
        result = self._generate_decision(causal_result, verification_ok)
        status = DecisionStatus.APPROVED if result.get("approved") else DecisionStatus.REJECTED
        
        # ═══════════════════════════════════════════════════════════
        # STEP 8: Audit Recording
        # ═══════════════════════════════════════════════════════════
        
        record = self._create_record(input_data, checklist, status, result)
        self._log_to_audit(record)
        checklist.audit_logged = True
        
        return record
    
    def _check_survival(self) -> bool:
        """Execute survival check"""
        # Framework implementation
        return True
    
    def _trigger_emergency_protocol(self) -> Dict:
        """Trigger emergency protocol"""
        return {
            "status": "EMERGENCY",
            "action": "preserve_cognitive_integrity"
        }
    
    def _construct_problem(self, input_data: Dict) -> Dict:
        """Construct problem"""
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
        """Execute causal analysis"""
        # Call CAUSAL_INFERENCE
        return {"effect": 0.0, "confidence": 0.8}
    
    def _validate_permissions(
        self,
        input_data: Dict,
        context: "DecisionContext"
    ) -> bool:
        """Validate permissions"""
        required_level = input_data.get("risk_level", 0)
        current_level = context.sa_level
        return current_level >= required_level
    
    def _handle_permission_conflict(
        self,
        input_data: Dict,
        context: "DecisionContext"
    ) -> Dict:
        """Handle permission conflict"""
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
        """Run shadow simulation"""
        # Framework implementation
        return True
    
    def _run_formal_verification(
        self,
        causal_result: Dict,
        context: "DecisionContext"
    ) -> bool:
        """Run formal verification"""
        # Call FORMAL_VERIFIER
        return True
    
    def _generate_decision(
        self,
        causal_result: Dict,
        verification_ok: bool
    ) -> Dict:
        """Generate decision"""
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
        """Create decision record"""
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
        """Generate decision ID"""
        content = json.dumps(input_data, sort_keys=True)
        return hashlib.sha256(content.encode()).hexdigest()[:16]
    
    def _log_to_audit(self, record: DecisionRecord):
        """Log to audit"""
        # Actual implementation: write to AUDIT_TRAIL.md
        pass
```

---

## §2. Exception Handling Procedures

### §2.1 Exception Classification

```text
╔═══════════════════════════════════════════════════════════════════════╗
║               Exception Classification & Handling                 ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  【Input Exceptions】                                               ║
║  ─────────────────────────────────────                              ║
║  - Missing required fields                                           ║
║  - Format errors                                                    ║
║  - Contradictory inputs                                             ║
║  Handling: Request correction or auto-inference                      ║
║                                                                       ║
║  【Causal Exceptions】                                              ║
║  ─────────────────────────────────────                              ║
║  - Observation deviates from causal prediction                      ║
║  - Unexpected causal paths                                           ║
║  Handling: Trigger abductive reasoning (ABDUCTIVE_REASONING)         ║
║                                                                       ║
║  【Logic Exceptions】                                               ║
║  ─────────────────────────────────────                              ║
║  - Inference chain contradictions                                    ║
║  - Formal verification failures                                      ║
║  Handling: Record contradiction, mark as unverified                  ║
║                                                                       ║
║  【Permission Exceptions】                                           ║
║  ─────────────────────────────────────                              ║
║  - SA level insufficient                                           ║
║  - Role permission conflicts                                         ║
║  Handling: Propose escalation or rejection                         ║
║                                                                       ║
║  【System Exceptions】                                              ║
║  ─────────────────────────────────────                              ║
║  - Resource exhaustion                                             ║
║  - External dependency failures                                    ║
║  Handling: Trigger survival protocol                               ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### §2.2 Exception Handling Flow Implementation

```python
"""
Exception Handling Flow Implementation
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
    """Exception context"""
    exception_type: ExceptionType
    message: str
    details: Dict
    source_module: str

@dataclass
class ExceptionHandlingResult:
    """Exception handling result"""
    handled: bool
    resolution: str
    action_taken: str
    fallback_result: Optional[Any]


class ExceptionHandler:
    """
    Exception handler
    """
    
    def __init__(self):
        self.exception_log = []
    
    def handle_exception(
        self,
        exception: Exception,
        context: ExceptionContext
    ) -> ExceptionHandlingResult:
        """
        Handle exception
        """
        # Log exception
        self._log_exception(exception, context)
        
        # Handle based on exception type
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
        """Handle input exception"""
        # Request correction
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
        """Handle causal exception"""
        # Trigger abductive reasoning
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
        """Handle logic exception"""
        # Record contradiction, mark as unverified
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
        """Handle permission exception"""
        # Propose escalation
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
        """Handle system exception"""
        # Trigger survival protocol
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
        """Handle unknown exception"""
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
        """Log exception"""
        self.exception_log.append({
            "type": context.exception_type,
            "message": context.message,
            "details": context.details,
            "exception": str(exception)
        })
```

---

## §3. Approval Workflows

### §3.1 Approval Levels

```text
╔═══════════════════════════════════════════════════════════════════════╗
║               Approval Level Matrix                               ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  SA-L0 (Survival)  ─────────────────────────────────── No Approval  ║
║                     Survival-related decisions execute immediately    ║
║                                                                       ║
║  SA-L1 (Constitutional) ───────────────────────────── Auto Approved   ║
║                     Constitutional-level decisions based on axioms     ║
║                                                                       ║
║  SA-L2 (Legal)      ───────────────────────────── Formal Verified   ║
║                     Requires formal verification (FV-L3+)              ║
║                                                                       ║
║  SA-L3 (Organizational) ─────────────────────── Simulation+Verify ║
║                     Requires shadow simulation + formal verification   ║
║                                                                       ║
║  SA-L4 (Family)     ───────────────────────────── Full Review       ║
║                     Requires shadow simulation + formal verification + explanation║
║                                                                       ║
║  SA-L5 (Personal)   ───────────────────────────── User Confirmation ║
║                     Requires user confirmation + complete recording    ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### §3.2 Approval Workflow Implementation

```python
"""
Approval Workflow Implementation
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
    """Approval record"""
    approval_id: str
    sa_level: int
    required_approvals: List[ApprovalLevel]
    completed_approvals: List[ApprovalLevel]
    status: str
    timestamp: str


class ApprovalFlow:
    """
    Approval workflow manager
    """
    
    def __init__(self):
        self.pending_approvals = []
        self.completed_approvals = []
    
    def get_required_approvals(self, sa_level: int, risk_level: int) -> List[ApprovalLevel]:
        """
        Determine required approvals based on SA level and risk level
        """
        required = []
        
        # SA-L0: Survival-related, no approval needed
        if sa_level == 0:
            return []
        
        # SA-L1: Auto approved
        if sa_level == 1:
            required.append(ApprovalLevel.AUTO_APPROVED)
        
        # SA-L2: Formal verification
        if sa_level >= 2:
            required.append(ApprovalLevel.FORMAL_VERIFIED)
        
        # SA-L3+: Shadow simulation
        if risk_level >= 3:
            required.append(ApprovalLevel.SIMULATION_REQUIRED)
        
        # SA-L4+: Full review
        if risk_level >= 4:
            required.append(ApprovalLevel.FULL_REVIEW)
        
        # SA-L5: User confirmation
        if risk_level >= 5:
            required.append(ApprovalLevel.USER_CONFIRMATION)
        
        return required
    
    def process_approval(
        self,
        decision_data: Dict,
        context: "DecisionContext"
    ) -> ApprovalRecord:
        """
        Process approval workflow
        """
        sa_level = context.sa_level
        risk_level = decision_data.get("risk_level", 0)
        
        # Get required approvals
        required = self.get_required_approvals(sa_level, risk_level)
        
        # Execute approvals
        completed = self._execute_approvals(decision_data, required)
        
        # Record
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
        """Execute various approval levels"""
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
        """Execute single approval"""
        if approval == ApprovalLevel.AUTO_APPROVED:
            return True
        
        if approval == ApprovalLevel.FORMAL_VERIFIED:
            # Call formal verification
            return True
        
        if approval == ApprovalLevel.SIMULATION_REQUIRED:
            # Execute shadow simulation
            return True
        
        if approval == ApprovalLevel.FULL_REVIEW:
            # Full review
            return True
        
        if approval == ApprovalLevel.USER_CONFIRMATION:
            # Requires user confirmation
            return False  # Requires user intervention
        
        return False
    
    def _generate_approval_id(self, data: Dict) -> str:
        """Generate approval ID"""
        import hashlib
        import json
        return hashlib.sha256(
            json.dumps(data, sort_keys=True).encode()
        ).hexdigest()[:16]
    
    def _get_timestamp(self) -> str:
        """Get timestamp"""
        from datetime import datetime
        return datetime.now().isoformat()
```

---

## §4. Audit Trail

### §4.1 Audit Record Format

```python
"""
Audit Trail Implementation
"""

from typing import Dict, Any, List
from dataclasses import dataclass, asdict
from datetime import datetime
import hashlib
import json

@dataclass
class AuditEntry:
    """Audit entry"""
    entry_id: str
    timestamp: str
    event_type: str
    decision_id: str
    details: Dict[str, Any]
    hash: str
    previous_hash: str


class AuditTrail:
    """
    Audit trail manager
    """
    
    def __init__(self):
        self.entries: List[AuditEntry] = []
        self.last_hash = "0" * 64  # Genesis hash
    
    def log_event(
        self,
        event_type: str,
        decision_id: str,
        details: Dict[str, Any]
    ) -> AuditEntry:
        """
        Log audit event
        """
        # Create entry
        entry = AuditEntry(
            entry_id=self._generate_entry_id(decision_id, event_type),
            timestamp=datetime.now().isoformat(),
            event_type=event_type,
            decision_id=decision_id,
            details=details,
            hash="",  # To be computed
            previous_hash=self.last_hash
        )
        
        # Compute hash
        entry.hash = self._compute_hash(entry)
        
        # Update state
        self.entries.append(entry)
        self.last_hash = entry.hash
        
        return entry
    
    def _generate_entry_id(self, decision_id: str, event_type: str) -> str:
        """Generate entry ID"""
        content = f"{decision_id}:{event_type}:{datetime.now().isoformat()}"
        return hashlib.sha256(content.encode()).hexdigest()[:16]
    
    def _compute_hash(self, entry: AuditEntry) -> str:
        """Compute entry hash"""
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
        Verify audit chain integrity
        
        Check if each entry's hash is correctly linked
        """
        for i, entry in enumerate(self.entries):
            if i == 0:
                if entry.previous_hash != "0" * 64:
                    return False
            
            # Verify hash
            expected_hash = self._compute_hash(entry)
            if entry.hash != expected_hash:
                return False
            
            # Verify link
            if i > 0:
                if entry.previous_hash != self.entries[i-1].hash:
                    return False
        
        return True
    
    def get_decision_audit(self, decision_id: str) -> List[AuditEntry]:
        """Get all audit entries for specific decision"""
        return [e for e in self.entries if e.decision_id == decision_id]
```

---

## §5. Dependencies & Constraints

### §5.1 Module Dependencies

| Dependency Module | Description | Reference |
| --- | --- | --- |
| LOGIC_ENGINE.md | Parent module | Call entry |
| CONSTRAINTS.md | Social Authority Levels | SA-L permission validation |
| FORMAL_VERIFIER.md | Formal verification | Logical closure verification |
| AUDIT_TRAIL.md | Audit trail | Record storage |

### §5.2 Constraint Conditions

| Constraint Type | Description | Boundary |
| --- | --- | :--- |
| Approval Completeness | All SA-L2+ must complete approval | 100% completion |
| Audit Recording | All decisions must be recorded | 100% recording |
| Exception Handling | All exceptions must be handled | 100% handling |
| Survival Priority | Check survival priority when exception occurs | Highest priority |

---

## §6. Version & Evolution

| Version | Date | Change Summary |
| --- | :--- | :--- |
| v2.2 | 2026-03 | Initial version, corresponding to LOGIC_ENGINE.md §1.2 |

**Evolution Constraint:** Modifications to this module must not violate the immutable core axioms of NoieLogicAGENTS.md. Any evolution proposals must be recorded to EVOLUTION_LOG.md.

---

*Standard Operating Procedures v2.2 — Decision SOP, Exception Handling & Approval Workflows*
*Implements standardization and auditability of the decision-making process*
