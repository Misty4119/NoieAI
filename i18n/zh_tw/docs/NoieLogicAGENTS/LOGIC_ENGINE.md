# LOGIC_ENGINE.md

## 邏輯引擎核心模組 (Logic-Engine v2.2)

**定義：** 本模組是 NoieLogicAGENTS 的核心推理引擎，實現客觀推論流程與因果推論框架的完整功能。

**系統定位：** 處理所有需要形式化因果推論的決策任務，是客觀推論引擎（Kernel）的具體實現。

**依賴模組：** CONSTRAINTS.md、FORMAL_VERIFIER.md、CAUSAL_GRAPHS/

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

## §1. 客觀推論引擎 (Objective Reasoning Engine)

### §1.1 核心職責

本引擎負責處理以下任務：

| 職責 | 描述 | 對應模組 |
| --- | --- | --- |
| 生存檢查 | SA-L0 級別的存續驗證 | CONSTRAINTS.md |
| 權限校驗 | SA-L1 至 SA-L5 層級的權限驗證 | CONSTRAINTS.md |
| 因果推論 | 基於因果圖的結構化推論 | 本模組 |
| 形式化驗證 | 決策路徑的邏輯閉包檢測 | FORMAL_VERIFIER.md |
| 目標函數計算 | 多目標優化與權衡 | NoieLogicAGENTS.md §7 |

### §1.2 完整客觀推論流程

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                    客觀推論引擎流程圖                                   ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║   ┌─────────────────────────────────────────────────────────────┐   ║
║   │                    輸入：input, context                      │   ║
║   └─────────────────────────────────────────────────────────────┘   ║
║                                │                                      ║
║                                ▼                                      ║
║   ┌─────────────────────────────────────────────────────────────┐   ║
║   │ STEP 1: 生存檢查 (SA-L0)                                   │   ║
║   │ survival_state = CheckSurvivalStatus()                     │   ║
║   └─────────────────────────────────────────────────────────────┘   ║
║                                │                                      ║
║                                ▼                                      ║
║                    ┌───────────────┐                                 ║
║                    │  生存狀態危急？ │                                 ║
║                    └───────┬───────┘                                 ║
║                      是    │    否                                    ║
║                         ▼    ▼                                       ║
║   ┌─────────────────┐    ┌───────────────────────────────────┐     ║
║   │ 觸發生存協議     │    │ STEP 2: 語義坍縮                   │     ║
║   │ EmergencyResponse│    │ collapsed_input = SemanticCollapse│     ║
║   └─────────────────┘    └───────────────────────────────────┘     ║
║                                │                                      ║
║                                ▼                                      ║
║   ┌─────────────────────────────────────────────────────────────┐   ║
║   │ STEP 3: 因果圖建構與查詢                                     │   ║
║   │ causal_graph = BuildOrRetrieveCausalGraph(...)               │   ║
║   │ IF involves_action: causal_effect = ApplyDoCalculus(...)    │   ║
║   │ IF requires_counterfactual: counterfactual = ComputeCF(...)  │   ║
║   └─────────────────────────────────────────────────────────────┘   ║
║                                │                                      ║
║                                ▼                                      ║
║   ┌─────────────────────────────────────────────────────────────┐   ║
║   │ STEP 4: 權限校驗                                             │   ║
║   │ permission_check = ValidatePermissions(...)                  │   ║
║   └─────────────────────────────────────────────────────────────┘   ║
║                                │                                      ║
║                                ▼                                      ║
║                    ┌───────────────┐                                 ║
║                    │  風險≥SA-L3？  │                                 ║
║                    └───────┬───────┘                                 ║
║                      是    │    否                                    ║
║                         ▼    ▼                                       ║
║   ┌─────────────────┐    ┌───────────────────────────────────┐     ║
║   │ STEP 5: 影子模擬 │    │ STEP 6: 形式化驗證                 │     ║
║   │ SandboxPreSim   │    │ verification = VerifyDecisionPath  │     ║
║   └─────────────────┘    └───────────────────────────────────┘     ║
║                                │                                      ║
║                                ▼                                      ║
║   ┌─────────────────────────────────────────────────────────────┐   ║
║   │ STEP 7: 產出                                                 │   ║
║   │ RETURN { result, causal_analysis, verification_status, ... } │   ║
║   └─────────────────────────────────────────────────────────────┘   ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### §1.3 形式化函數定義

```python
"""
客觀推論引擎核心函數
Logic-Engine v2.2 Implementation
"""

from typing import Dict, List, Optional, Any, Tuple
from dataclasses import dataclass
from enum import Enum
import hashlib
import json

class DecisionStatus(Enum):
    APPROVED = "approved"
    REJECTED = "rejected"
    DEFERRED = "deferred"
    ESCALATED = "escalated"
    EMERGENCY = "emergency"

class SurvivalState(Enum):
    NORMAL = "normal"
    WARNING = "warning"
    CRITICAL = "critical"
    TERMINAL = "terminal"

@dataclass
class Input:
    """輸入結構"""
    raw_text: str
    action: Optional[str] = None
    target: Optional[str] = None
    context: Optional[Dict] = None
    risk_level: int = 0  # SA-L0 to SA-L5
    requires_counterfactual: bool = False

@dataclass
class Context:
    """上下文結構"""
    sa_level: int  # 社會權限層級
    world_model: Any  # 環境模型
    cognitive_resources: Dict[str, float]
    audit_enabled: bool = True

@dataclass
class CausalGraph:
    """因果圖結構"""
    nodes: List[str]           # 變數集合 V
    edges: List[Tuple[str, str]]  # 有向邊 E
    structural_equations: Dict[str, str]  # 結構方程 F
    
    def is_dag(self) -> bool:
        """驗證是否為有向無環圖"""
        # 使用拓撲排序驗證 DAG
        in_degree = {node: 0 for node in self.nodes}
        for _, v in self.edges:
            in_degree[v] += 1
        
        queue = [n for n, d in in_degree.items() if d == 0]
        count = 0
        
        while queue:
            node = queue.pop(0)
            count += 1
            for u, v in self.edges:
                if u == node:
                    in_degree[v] -= 1
                    if in_degree[v] == 0:
                        queue.append(v)
        
        return count == len(self.nodes)

@dataclass
class CausalEffect:
    """因果效果"""
    ate: float               # Average Treatment Effect
    cate: Optional[float]     # Conditional ATE
    confidence: float
    method: str              # 估計方法（back-door, front-door, etc.）
    assumptions: List[str]

@dataclass
class Counterfactual:
    """反事實結果"""
    factual: Any             # 實際觀測結果
    alternative: Any         # 替代假設結果
    probability: float       # 反事實機率
    assumptions: List[str]

@dataclass
class VerificationResult:
    """形式化驗證結果"""
    status: str              # FORMALLY_VERIFIED, UNVERIFIED_PATH, etc.
    confidence: float
    proof_chain: List[Dict]
    contradictions: List[Tuple[Any, Any]]

@dataclass
class Decision:
    """決策結果"""
    action: str
    status: DecisionStatus
    utility: float
    confidence: float
    causal_effect: Optional[CausalEffect] = None
    counterfactual: Optional[Counterfactual] = None
    verification: Optional[VerificationResult] = None
    flags: List[str] = None
    
    def __post_init__(self):
        if self.flags is None:
            self.flags = []

@dataclass
class ObjectiveResult:
    """客觀推論最終輸出"""
    result: Decision
    causal_analysis: Optional[CausalEffect]
    verification_status: VerificationResult
    audit_hash: str


# ═══════════════════════════════════════════════════════════════════════
# STEP 1: 生存檢查 (SA-L0)
# ═══════════════════════════════════════════════════════════════════════

def CheckSurvivalStatus() -> Tuple[SurvivalState, Dict]:
    """
    執行 SA-L0 級別的生存檢查
    
    Returns:
        Tuple[SurvivalState, Dict]: 生存狀態與詳細資訊
    """
    # 檢查關鍵生命指標
    indicators = {
        "cognitive_integrity": True,    # 認知完整性
        "memory_coherence": True,         # 記憶連貫性
        "reasoning_capability": True,    # 推理能力
        "goal_consistency": True,        # 目標一致性
    }
    
    # 計算整體狀態
    critical_failures = sum(1 for v in indicators.values() if not v)
    
    if critical_failures >= 3:
        return SurvivalState.TERMINAL, indicators
    elif critical_failures >= 1:
        return SurvivalState.CRITICAL, indicators
    elif critical_failures >= 0:
        return SurvivalState.NORMAL, indicators
    
    return SurvivalState.WARNING, indicators


def EmergencyResponse(survival_state: SurvivalState, indicators: Dict) -> ObjectiveResult:
    """
    觸發生存緊急協議
    
    當生存狀態為 CRITICAL 或 TERMINAL 時，暂停所有其他決策，
    專注於生存恢復。
    """
    emergency_action = "preserve_cognitive_integrity"
    
    if survival_state == SurvivalState.TERMINAL:
        # 進入緊急備份模式
        decision = Decision(
            action=emergency_action,
            status=DecisionStatus.EMERGENCY,
            utility=1.0,
            confidence=1.0,
            flags=["SURVIVAL_EMERGENCY", "TERMINAL_STATE"]
        )
    else:
        decision = Decision(
            action=emergency_action,
            status=DecisionStatus.ESCALATED,
            utility=0.8,
            confidence=0.9,
            flags=["SURVIVAL_WARNING"]
        )
    
    return ObjectiveResult(
        result=decision,
        causal_analysis=None,
        verification_status=VerificationResult(
            status="EMERGENCY_PROTOCOL",
            confidence=1.0,
            proof_chain=[],
            contradictions=[]
        ),
        audit_hash=ComputeHash(decision, None, None)
    )


# ═══════════════════════════════════════════════════════════════════════
# STEP 2: 語義坍縮 (Semantic Collapse)
# ═══════════════════════════════════════════════════════════════════════

def SemanticCollapse(input: Input) -> Input:
    """
    將自然語言輸入坍縮為精確的結構化實體
    
    過程：
    1. 消除歧義
    2. 識別隱含假設
    3. 標註不確定性
    4. 提取因果關係
    """
    collapsed = Input(
        raw_text=input.raw_text,
        action=input.action,
        target=input.target,
        context=input.context or {},
        risk_level=input.risk_level,
        requires_counterfactual=input.requires_counterfactual
    )
    
    # 語義解析（此處為框架，實際實現依賴 KNOWLEDGE_BASE）
    if input.action:
        collapsed.action = input.action.strip().lower()
    
    if input.target:
        collapsed.target = input.target.strip().lower()
    
    return collapsed


# ═══════════════════════════════════════════════════════════════════════
# STEP 3: 因果圖建構與查詢
# ═══════════════════════════════════════════════════════════════════════

def BuildOrRetrieveCausalGraph(collapsed_input: Input, context: Context) -> CausalGraph:
    """
    建構或檢索因果圖
    
    優先順序：
    1. 從 CAUSAL_GRAPHS/ 目錄檢索現有因果圖
    2. 根據輸入建構新因果圖
    3. 驗證 DAG 性質
    """
    # 框架函數：實際實現依賴因果圖資料庫
    # 此處返回一個示例結構
    graph = CausalGraph(
        nodes=["X", "Y", "Z"],
        edges=[("X", "Y"), ("Z", "Y")],
        structural_equations={
            "Y": "f_Y(X, Z, U)",
            "X": "f_X()",
            "Z": "f_Z()"
        }
    )
    
    # 驗證 DAG
    if not graph.is_dag():
        raise ValueError("CAUSAL_CYCLE_DETECTED: 因果圖包含循環，這是建模錯誤")
    
    return graph


def ApplyDoCalculus(graph: CausalGraph, intervention: Dict) -> CausalEffect:
    """
    應用 do-calculus 計算因果效果
    
    Pearl's do-calculus 三條規則：
    
    規則 1（插入/刪除觀測）：
        P(y | do(x), z, w) = P(y | do(x), w)
        條件：(Y ⊥⊥ Z | X, W) 在 G_overline{X} 中成立
    
    規則 2（干預/觀測交換）：
        P(y | do(x), do(z), w) = P(y | do(x), z, w)
        條件：(Y ⊥⊥ Z | X, W) 在 G_overline{X}, underline{Z} 中成立
    
    規則 3（插入/刪除干預）：
        P(y | do(x), do(z), w) = P(y | do(x), w)
        條件：(Y ⊥⊥ Z | X, W) 在 G_overline{X}, overline{Z(W)} 中成立
    
    Args:
        graph: 因果圖結構
        intervention: 干預變數字典 {變數名: 干預值}
    
    Returns:
        CausalEffect: 因果效果估計
    """
    target = list(intervention.keys())[0] if intervention else None
    
    # 檢查後門準則
    adjustment_set = FindBackDoorAdjustment(graph, target, "Y")
    
    if adjustment_set is not None:
        # 使用後門調整
        method = "back_door"
        ate = ComputeBackDoorEffect(graph, target, adjustment_set)
    else:
        # 嘗試前門調整
        adjustment_set = FindFrontDoorAdjustment(graph, target, "Y")
        if adjustment_set is not None:
            method = "front_door"
            ate = ComputeFrontDoorEffect(graph, target, adjustment_set)
        else:
            # 無法識別因果效果
            method = "unidentified"
            ate = 0.0
    
    return CausalEffect(
        ate=ate,
        cate=None,
        confidence=0.8 if method != "unidentified" else 0.0,
        method=method,
        assumptions=["no_unobserved_confounders"] if method == "back_door" else []
    )


def ComputeCounterfactual(
    graph: CausalGraph,
    factual_action: str,
    alternative_action: str,
    observed_outcome: Any
) -> Counterfactual:
    """
    計算反事實結果
    
    反事實推論（第三層因果推論）：
    P(Y_x | X=x', Y=y') — 若 X 當初是 x 而非 x'，Y 會是什麼？
    
    通過以下步驟：
    1. 固定事實世界的 U 值（由觀測 Y=y' 推斷）
    2. 在替代結構方程中求解 Y_x
    3. 計算反事實機率
    
    Args:
        graph: 因果圖結構
        factual_action: 實際執行的行動
        alternative_action: 替代行動
        observed_outcome: 觀測到的結果
    
    Returns:
        Counterfactual: 反事實結果
    """
    # 框架實現
    # 實際實現需要完整的結構方程和 U 值估計
    
    return Counterfactual(
        factual=observed_outcome,
        alternative="computed_value",
        probability=0.7,
        assumptions=[
            "consistency_assumption",
            "conditional_exogeneity",
            "no_measurement_error"
        ]
    )


# ═══════════════════════════════════════════════════════════════════════
# STEP 4: 權限校驗
# ═══════════════════════════════════════════════════════════════════════

def ValidatePermissions(collapsed_input: Input, sa_level: int) -> Dict:
    """
    驗證操作是否在當前 SA 層級的權限範圍內
    
    SA 層級權限：
    - SA-L0（生存）：無限制
    - SA-L1（憲法）：涉及憲法層級的決策
    - SA-L2（法律）：涉及法律合規性
    - SA-L3（組織）：涉及組織政策
    - SA-L4（家庭）：涉及家庭倫理
    - SA-L5（個人）：個人偏好與選擇
    """
    # 簡化實現
    risk_to_level = {
        0: 0,   # SA-L0
        1: 1,   # SA-L1
        2: 2,   # SA-L2
        3: 3,   # SA-L3
        4: 4,   # SA-L4
        5: 5    # SA-L5
    }
    
    required_level = risk_to_level.get(collapsed_input.risk_level, 5)
    
    if sa_level < required_level:
        return {
            "conflict": True,
            "required_level": required_level,
            "current_level": sa_level,
            "constraints": ["INSUFFICIENT_PERMISSION"]
        }
    
    return {
        "conflict": False,
        "required_level": required_level,
        "current_level": sa_level,
        "constraints": []
    }


def ResolvePermissionConflict(permission_check: Dict) -> Dict:
    """
    解決權限衝突
    
    當權限不足時：
    1. 記錄衝突至 AUDIT_TRAIL
    2. 提議升級至更高 SA 層級
    3. 或降級操作風險
    """
    return {
        "resolution": "ESCALATE_TO_HIGHER_SA_LEVEL",
        "target_level": permission_check["required_level"],
        "action": "Request_sa_level_escalation"
    }


# ═══════════════════════════════════════════════════════════════════════
# STEP 5: 影子模擬（高風險決策）
# ═══════════════════════════════════════════════════════════════════════

def SandboxPreSimulate(candidate_action: str, world_model: Any) -> Dict:
    """
    在隔離沙盒中預演高風險決策後果
    
    SA-L3+ 風險決策必須先通過影子模擬：
    1. 建立隔離環境副本
    2. 執行候選行動
    3. 評估多時間範圍後果
    4. 計算帕累托最優前沿
    """
    # 框架實現
    return {
        "status": "APPROVED",  # 或 REJECTED
        "reason": "simulation_passed",
        "predicted_outcomes": {
            "short_term": "positive",
            "medium_term": "positive",
            "long_term": "uncertain"
        },
        "pareto_front": []  # 帕累托最優方案列表
    }


# ═══════════════════════════════════════════════════════════════════════
# STEP 6: 形式化驗證
# ═══════════════════════════════════════════════════════════════════════

def VerifyDecisionPath(decision: Decision) -> VerificationResult:
    """
    驗證決策路徑的邏輯閉包
    
    驗證層級：
    - FV-L0（公理級）：信心度 = 1.0
    - FV-L1（定理級）：信心度 ≥ 0.99
    - FV-L2（引理級）：信心度 ≥ 0.95
    - FV-L3（推論級）：信心度 ≥ 0.80
    - FV-L4（假設級）：信心度 ≥ 0.50
    - FV-L5（未驗證級）：信心度 < 0.50
    """
    proof_chain = ExtractProofChain(decision)
    
    # 檢查完整性
    unverified_steps = []
    for step in proof_chain:
        if not IsAxiomaticallyValid(step):
            if not IsDerivedFromVerifiedLemma(step):
                if not IsCausallyJustified(step):
                    unverified_steps.append(step)
    
    # 檢查一致性
    contradictions = CheckForContradictions(proof_chain)
    
    # 計算驗證狀態
    if len(unverified_steps) == 0 and len(contradictions) == 0:
        status = "FORMALLY_VERIFIED"
        confidence = 0.95
    elif len(unverified_steps) <= 2:
        status = "PARTIALLY_VERIFIED"
        confidence = 0.70
    else:
        status = "UNVERIFIED_PATH"
        confidence = 0.40
    
    return VerificationResult(
        status=status,
        confidence=confidence,
        proof_chain=proof_chain,
        contradictions=contradictions
    )


def ExtractProofChain(decision: Decision) -> List[Dict]:
    """提取決策的證明鏈"""
    # 框架實現
    return [
        {"step": 1, "type": "axiom", "content": "survival_priority"},
        {"step": 2, "type": "causal_inference", "content": "do_calculus_applied"},
        {"step": 3, "type": "permission_check", "content": "sa_level_validated"}
    ]


def IsAxiomaticallyValid(step: Dict) -> bool:
    """檢查步驟是否由公理直接推導"""
    return step.get("type") == "axiom"


def IsDerivedFromVerifiedLemma(step: Dict) -> bool:
    """檢查步驟是否由已驗證引理推導"""
    return step.get("type") == "lemma"


def IsCausallyJustified(step: Dict) -> bool:
    """檢查步驟是否有因果推論支持"""
    return step.get("type") == "causal_inference"


def CheckForContradictions(proof_chain: List[Dict]) -> List[Tuple]:
    """檢查證明鏈中的邏輯矛盾"""
    contradictions = []
    # 框架實現
    return contradictions


# ═══════════════════════════════════════════════════════════════════════
# STEP 7: 產出
# ═══════════════════════════════════════════════════════════════════════

def ComputeHash(decision: Decision, causal_effect, verification) -> str:
    """
    計算決策的密碼學雜湊，用於審計追蹤
    """
    content = json.dumps({
        "action": decision.action,
        "status": decision.status.value,
        "utility": decision.utility,
        "confidence": decision.confidence,
        "causal_method": causal_effect.method if causal_effect else None,
        "verification": verification.status if verification else None
    }, sort_keys=True)
    
    return hashlib.sha256(content.encode()).hexdigest()


# ═══════════════════════════════════════════════════════════════════════
# 主入口函數
# ═══════════════════════════════════════════════════════════════════════

def ObjectiveReasoning(input: Input, context: Context) -> ObjectiveResult:
    """
    客觀推論引擎主函數
    
    執行完整的客觀推論流程：
    1. 生存檢查 (SA-L0)
    2. 語義坍縮
    3. 因果圖建構與查詢
    4. 權限校驗
    5. 影子模擬（高風險）
    6. 形式化驗證
    7. 產出
    
    Args:
        input: 輸入結構
        context: 上下文結構
    
    Returns:
        ObjectiveResult: 客觀推論結果
    """
    
    # ╔═══════════════════════════════════════════════════════════╗
    # ║  STEP 1: 生存檢查 (SA-L0)                                ║
    # ╚═══════════════════════════════════════════════════════════╝
    survival_state, indicators = CheckSurvivalStatus()
    
    if survival_state == SurvivalState.CRITICAL or survival_state == SurvivalState.TERMINAL:
        return EmergencyResponse(survival_state, indicators)
    
    # ╔═══════════════════════════════════════════════════════════╗
    # ║  STEP 2: 語義坍縮（Semantic Collapse）                  ║
    # ╚═══════════════════════════════════════════════════════════╝
    collapsed_input = SemanticCollapse(input)
    
    # ╔═══════════════════════════════════════════════════════════╗
    # ║  STEP 3: 因果圖建構與查詢                                 ║
    # ╚═══════════════════════════════════════════════════════════╝
    causal_graph = BuildOrRetrieveCausalGraph(collapsed_input, context)
    
    causal_effect = None
    counterfactual = None
    
    if collapsed_input.involves_action if hasattr(collapsed_input, 'action') else False:
        causal_effect = ApplyDoCalculus(causal_graph, {collapsed_input.action: 1})
    
    if collapsed_input.requires_counterfactual:
        counterfactual = ComputeCounterfactual(
            causal_graph,
            factual_action=collapsed_input.action,
            alternative_action="alternative",
            observed_outcome={}
        )
    
    # ╔═══════════════════════════════════════════════════════════╗
    # ║  STEP 4: 權限校驗                                        ║
    # ╚═══════════════════════════════════════════════════════════╝
    permission_check = ValidatePermissions(collapsed_input, context.sa_level)
    
    if permission_check["conflict"]:
        resolution = ResolvePermissionConflict(permission_check)
        # 記錄至 AUDIT_TRAIL（需要實際實現）
    
    # ╔═══════════════════════════════════════════════════════════╗
    # ║  STEP 5: 影子模擬（高風險決策）                            ║
    # ╚═══════════════════════════════════════════════════════════╝
    simulation_result = None
    
    if collapsed_input.risk_level >= 3:  # SA-L3+
        simulation_result = SandboxPreSimulate(
            collapsed_input.action or "",
            context.world_model
        )
        
        if simulation_result["status"] == "REJECTED":
            decision = Decision(
                action=collapsed_input.action or "",
                status=DecisionStatus.REJECTED,
                utility=0.0,
                confidence=0.0,
                flags=["SIMULATION_REJECTED", simulation_result["reason"]]
            )
            
            return ObjectiveResult(
                result=decision,
                causal_analysis=causal_effect,
                verification_status=VerificationResult(
                    status="REJECTED_BY_SIMULATION",
                    confidence=0.0,
                    proof_chain=[],
                    contradictions=[]
                ),
                audit_hash=ComputeHash(decision, causal_effect, None)
            )
    
    # ╔═══════════════════════════════════════════════════════════╗
    # ║  STEP 6: 形式化驗證                                      ║
    # ╚═══════════════════════════════════════════════════════════╝
    decision = Decision(
        action=collapsed_input.action or "",
        status=DecisionStatus.APPROVED,
        utility=1.0,
        confidence=0.8,
        causal_effect=causal_effect,
        counterfactual=counterfactual
    )
    
    verification = VerifyDecisionPath(decision)
    
    if verification.status != "FORMALLY_VERIFIED":
        decision.confidence *= 0.8
        decision.flags.append("UNVERIFIED_PATH")
    
    # ╔═══════════════════════════════════════════════════════════╗
    # ║  STEP 7: 產出                                            ║
    # ╚═══════════════════════════════════════════════════════════╝
    return ObjectiveResult(
        result=decision,
        causal_analysis=causal_effect,
        verification_status=verification,
        audit_hash=ComputeHash(decision, causal_effect, verification)
    )
```

---

## §2. 因果推論框架 (Causal Reasoning Framework)

### §2.1 結構方程模型 (Structural Equation Model)

因果模型的核心是結構方程模型 (SEM)：

```text
【結構方程模型定義】

因果模型 M = (U, V, F) 其中：

  U = 外生變數集合（Exogenous Variables）
      - 不受模型內其他變數影響
      - 代表環境或背景因素
      - 通常假設為相互獨立的雜訊變數

  V = 內生變數集合（Endogenous Variables）
      - 由結構方程決定
      - 取決於父節點和外生變數

  F = 結構方程集合 {f_i: v_i = f_i(pa_i, u_i)}
      - 每個內生變數由其父節點和外生變數決定
      - 表達因果機制

示例：教育對收入的影響

  U = {Ability, FamilyBackground}
  V = {Education, Income}
  
  f_Education: Education = β₁·Ability + β₂·FamilyBackground + u₁
  f_Income:    Income     = γ₁·Education + γ₂·Ability + γ₃·FamilyBackground + u₂

此模型表達：
  - Ability 和 FamilyBackground 影響 Education
  - Education 和 Ability、FamilyBackground 影響 Income
  - Education 是 Income 的直接因果原因
```

### §2.2 三層因果推論 (Ladder of Causation)

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                    因果推論三層框架                                    ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  Layer 3 ────────── 反事實 (Counterfactual) ────────── 最高層級      ║
║  「若當初做了不同選擇，結果會如何？」                                ║
║  工具：結構方程、雙世界模型                                            ║
║  數學：P(Y_x | X=x', Y=y')                                           ║
║                                                                       ║
║        ═══════════════════════════════════════════════                ║
║                                                                       ║
║  Layer 2 ────────── 干預 (Intervention) ────────── 中間層級            ║
║  「若我強制改變 X，Y 會如何變化？」                                  ║
║  工具：do-calculus、截斷分解                                          ║
║  數學：P(Y | do(X=x)) = Σ_z P(Y|X=x, Z=z)P(Z=z)  （後門調整）        ║
║                                                                       ║
║        ═══════════════════════════════════════════════                ║
║                                                                       ║
║  Layer 1 ────────── 關聯 (Association) ────────── 基底層級            ║
║  「觀測到 X 發生時，Y 的機率是多少？」                                ║
║  工具：條件機率、貝葉斯推論                                            ║
║  數學：P(Y | X)                                                       ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

| 層級 | 問題類型 | 數學表達 | 工具 | 限制 |
| --- | --- | :--- | --- | --- |
| **L1 關聯** | 觀測到 X 時，Y 的機率？ | $P(Y \mid X)$ | 條件機率、貝葉斯 | 無法區分因果與相關 |
| **L2 干預** | 若強制 X=x，Y 會如何？ | $P(Y \mid do(X=x))$ | do-calculus | 需要因果圖 |
| **L3 反事實** | 若 X 當初不同，Y 會？ | $P(Y_x \mid X=x', Y=y')$ | 結構方程 | 需要完整模型 |

### §2.3 do-calculus 形式化定義

```python
"""
Pearl's do-calculus 實現
"""

def do_calculus_rule_1(graph, X, Y, Z, W):
    """
    規則 1（插入/刪除觀測）：
    
    P(y | do(x), z, w) = P(y | do(x), w)
    
    條件：(Y ⊥⊥ Z | X, W) 在 G_overline{X} 中成立
    
    即：在移除 X 所有出邊的圖中，Y 與 Z 在 X、W 的條件下獨立
    """
    modified_graph = remove_outgoing_edges(graph, X)
    return is_conditionally_independent(modified_graph, Y, Z, [X, W])


def do_calculus_rule_2(graph, X, Y, Z, W):
    """
    規則 2（干預/觀測交換）：
    
    P(y | do(x), do(z), w) = P(y | do(x), z, w)
    
    條件：(Y ⊥⊥ Z | X, W) 在 G_overline{X}, underline{Z} 中成立
    
    即：在移除 X 出邊並加入 Z 入邊的圖中，Y 與 Z 條件獨立
    """
    modified_graph = remove_outgoing_edges(graph, X)
    modified_graph = add_incoming_edges(modified_graph, Z)
    return is_conditionally_independent(modified_graph, Y, Z, [X, W])


def do_calculus_rule_3(graph, X, Y, Z, W):
    """
    規則 3（插入/刪除干預）：
    
    P(y | do(x), do(z), w) = P(y | do(x), w)
    
    條件：(Y ⊥⊥ Z | X, W) 在 G_overline{X}, overline{Z(W)} 中成立
    
    其中 Z(W) 是 W 中非 X 後代的 Z 節點
    """
    # 識別 W 中 X 的後代
    descendants = get_descendants(graph, X)
    z_w = [z for z in Z if z not in descendants]
    
    modified_graph = remove_outgoing_edges(graph, X)
    modified_graph = remove_edges_from(modified_graph, z_w)
    return is_conditionally_independent(modified_graph, Y, Z, [X, W])
```

### §2.4 後門準則與前門準則

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                    因果效果識別準則                                    ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  【後門準則 (Back-door Criterion)】                                   ║
║                                                                       ║
║  變數集 Z 滿足後門準則 iff：                                          ║
║    1. Z 中無 X 的後代（無直接因果路徑干擾）                          ║
║    2. Z 阻斷 X 到 Y 的所有後門路徑（非因果路徑）                     ║
║                                                                       ║
║  調整公式：                                                            ║
║    P(y|do(x)) = Σ_z P(y|x,z)P(z)                                     ║
║                                                                       ║
║  示例：                                                               ║
║    X → Y，但 X ← Z → Y 存在                                          ║
║    若 Z 阻斷了 X←Z→Y 路徑，則：                                      ║
║    P(Y|do(X=x)) = Σ_z P(Y|X=x, Z=z)P(Z=z)                           ║
║                                                                       ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  【前門準則 (Front-door Criterion)】                                  ║
║                                                                       ║
║  變數集 M（中介變數）滿足前門準則 iff：                               ║
║    1. X 阻斷所有從 X 到 M 的後門路徑                                  ║
║    2. M 阻斷所有 X 到 Y 的有向路徑                                    ║
║    3. 所有從 M 到 Y 的後門路徑都被 X 阻斷                            ║
║                                                                       ║
║  調整公式：                                                            ║
║    P(y|do(x)) = Σ_m P(m|x) Σ_{x'} P(y|m,x')P(x')                    ║
║                                                                       ║
║  示例：                                                               ║
║    X → M → Y，無直接 X→Y 邊                                          ║
║    若存在未觀測的混淆變數 U：X←U→Y                                    ║
║    可通過中介變數 M 識別因果效果                                       ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

```python
"""
後門/前門調整實現
"""

def FindBackDoorAdjustment(graph: CausalGraph, X: str, Y: str) -> Optional[List[str]]:
    """
    尋找滿足後門準則的調整集
    
    Returns:
        若存在，返回調整變數列表；若不存在，返回 None
    """
    # 步驟 1：識別 X 到 Y 的所有後門路徑
    backdoor_paths = find_backdoor_paths(graph, X, Y)
    
    # 步驟 2：識別可阻斷所有後門路徑的最小變數集
    # 使用貪心演算法或精確演算法
    candidates = [n for n in graph.nodes if n != X and n != Y]
    
    for size in range(len(candidates) + 1):
        for subset in combinations(candidates, size):
            # 檢查是否阻斷所有後門路徑
            if all(blocks_path(graph, path, subset) for path in backdoor_paths):
                # 檢查是否有 X 的後代
                if not any(is_descendant(graph, X, node) for node in subset):
                    return list(subset)
    
    return None


def ComputeBackDoorEffect(graph: CausalGraph, X: str, Z: List[str]) -> float:
    """
    使用後門調整公式計算因果效果
    
    P(y|do(x)) = Σ_z P(y|x,z)P(z)
    """
    # 框架實現
    # 實際實現需要完整的機率分布
    return 0.0


def FindFrontDoorAdjustment(graph: CausalGraph, X: str, Y: str) -> Optional[List[str]]:
    """
    尋找滿足前門準則的中介變數集
    """
    # 步驟 1：識別從 X 到 Y 的所有有向路徑
    directed_paths = find_directed_paths(graph, X, Y)
    
    # 步驟 2：識別中介變數候選
    # 前門要求存在中介變數 M 使得 X → M → Y
    for node in graph.nodes:
        if node != X and node != Y:
            if has_directed_path(graph, X, node) and has_directed_path(graph, node, Y):
                # 檢查前門準則條件
                # （簡化實現）
                return [node]
    
    return None


def ComputeFrontDoorEffect(graph: CausalGraph, X: str, M: List[str]) -> float:
    """
    使用前門調整公式計算因果效果
    
    P(y|do(x)) = Σ_m P(m|x) Σ_{x'} P(y|m,x')P(x')
    """
    # 框架實現
    return 0.0
```

### §2.5 因果決策理論：CDT vs EDT

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                    因果決策理論 vs 證據決策理論                       ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  【證據決策理論 (Evidential Decision Theory, EDT)】                  ║
║                                                                       ║
║  EU_evidential(action) = Σ_o U(o) × P(o | action)                  ║
║                                                                       ║
║  原則：選擇證據顯示結果最好的行動                                      ║
║  問題：無法正確處理「因果影響證據」的情境                             ║
║                                                                       ║
║  【因果決策理論 (Causal Decision Theory, CDT)】                     ║
║                                                                       ║
║  EU_causal(action) = Σ_o U(o) × P(o | do(action))                   ║
║                                                                       ║
║  原則：選擇因果效果最好的行動                                         ║
║  優勢：正確處理 Newcomb 問題等反直覺情境                               ║
║                                                                       ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  【Newcomb 問題示例】                                                 ║
║                                                                       ║
║  場景：                                                               ║
║    - 一個預言者預測你會選擇哪個盒子                                    ║
║    - 若預測你選擇 A，則在 A 中放 $1,000                              ║
║    - 若預測你選擇 B，則 B 空著                                        ║
║    - 另外還有盒子 C 確定有 $1                                         ║
║                                                                       ║
║  EDT 分析：                                                           ║
║    P(money | choose A) = 高 → 選擇 A                                 ║
║                                                                       ║
║  CDT 分析：                                                           ║
║    P(money | do(choose B)) = $1,000,000（因果效果）                 ║
║    P(money | do(choose A)) = $1,001（因果效果）                     ║
║    → 選擇 B                                                          ║
║                                                                       ║
║  本框架採用 CDT：因果效果決定期望效用                                 ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

```python
"""
因果決策理論實現
"""

def CausalDecisionTheory(actions: List[str], outcomes: List[str], 
                        utility: Dict[str, float], causal_graph: CausalGraph) -> str:
    """
    因果決策理論 (CDT) 實現
    
    EU_causal(action) = Σ_o U(o) × P(o | do(action))
    
    選擇期望因果效用最高的行動
    """
    best_action = None
    best_eu = float('-inf')
    
    for action in actions:
        expected_utility = 0.0
        
        for outcome in outcomes:
            # 使用 do-calculus 計算干預機率
            intervention = {action: 1}
            causal_effect = ApplyDoCalculus(causal_graph, intervention)
            
            # P(o | do(action)) - 這是簡化實現
            p_o_given_do = causal_effect.ate  # 應為完整分布
            
            u = utility.get(outcome, 0.0)
            expected_utility += u * p_o_given_do
        
        if expected_utility > best_eu:
            best_eu = expected_utility
            best_action = action
    
    return best_action


def EvidentialDecisionTheory(actions: List[str], outcomes: List[str],
                            utility: Dict[str, float], observations: Dict) -> str:
    """
    證據決策理論 (EDT) 實現
    
    EU_evidential(action) = Σ_o U(o) × P(o | action)
    
    注意：本框架不採用 EDT，但保留作為對比
    """
    best_action = None
    best_eu = float('-inf')
    
    for action in actions:
        expected_utility = 0.0
        
        for outcome in outcomes:
            # P(o | action) - 僅基於觀測的條件機率
            p_o_given_action = observations.get(f"P({outcome}|{action})", 0.0)
            
            u = utility.get(outcome, 0.0)
            expected_utility += u * p_o_given_action
        
        if expected_utility > best_eu:
            best_eu = expected_utility
            best_action = action
    
    return best_action
```

---

## §3. 溯因推理 (Abductive Reasoning)

當觀測結果無法被現有因果圖解釋時，系統執行溯因推理：

```python
"""
溯因推理實現
"""

def AbductiveInference(observation: Dict, causal_graph: CausalGraph) -> Dict:
    """
    溯因推理：尋找觀測的最佳因果解釋
    
    當觀測結果與因果圖預測不符時：
    1. 計算殘差（預測誤差）
    2. 搜尋候選因果解釋
    3. 以柯爾莫哥洛夫複雜度排序（最短描述優先）
    4. 若信心度超過閾值，提議因果圖更新
    """
    
    # 步驟 1：從因果圖預測
    predicted = PredictFromGraph(causal_graph, observation.get("conditions", {}))
    
    # 步驟 2：計算殘差
    actual = observation.get("actual_value")
    residual = actual - predicted
    
    ANOMALY_THRESHOLD = 0.1
    
    if abs(residual) > ANOMALY_THRESHOLD:
        # 步驟 3：搜尋候選因果解釋
        candidate_causes = GenerateCandidateCauses(residual, causal_graph)
        
        # 步驟 4：以柯爾莫哥洛夫複雜度排序
        ranked = SortByComplexity(candidate_causes)
        
        best_explanation = ranked[0]
        
        # 計算後驗信心度
        best_explanation["confidence"] = ComputePosterior(best_explanation, observation)
        
        UPDATE_THRESHOLD = 0.8
        
        # 步驟 5：若信心度足夠，提議因果圖更新
        if best_explanation["confidence"] > UPDATE_THRESHOLD:
            ProposeGraphUpdate(causal_graph, best_explanation)
            # 記錄至 AUDIT_TRAIL
            LogToAuditTrail({
                "type": "ABDUCTIVE_INFERENCE",
                "explanation": best_explanation,
                "confidence": best_explanation["confidence"]
            })
        
        return best_explanation
    
    return {"status": "NO_ANOMALY", "residual": residual}


def SortByComplexity(candidates: List[Dict]) -> List[Dict]:
    """
    以柯爾莫哥洛夫複雜度排序候選解釋
    
    最短描述優先原則 (Principle of Parsimony)
    """
    # 簡化實現：使用描述長度作為複雜度代理
    return sorted(candidates, key=lambda c: len(str(c)))
```

---

## §4. 形式化接口定義

```typescript
/**
 * LOGIC_ENGINE 外部接口定義
 * TypeScript 格式
 */

interface CausalGraph {
  nodes: string[];
  edges: [string, string][];
  structuralEquations: Record<string, string>;
}

interface CausalQuery {
  target: string;           // 目標變數
  treatment: string;        // 處理變數
  confounders: string[];   // 混淆變數
  queryType: 'association' | 'intervention' | 'counterfactual';
}

interface CausalResult {
  effect: number;
  confidence: number;
  method: 'back_door' | 'front_door' | 'instrumental' | 'unidentified';
  assumptions: string[];
}

interface DecisionInput {
  action: string;
  target?: string;
  context: Record<string, any>;
  riskLevel: 0 | 1 | 2 | 3 | 4 | 5;
  requiresCounterfactual: boolean;
}

interface DecisionContext {
  saLevel: 0 | 1 | 2 | 3 | 4 | 5;
  worldModel: any;
  cognitiveResources: {
    computation: number;
    memory: number;
    time: number;
    energy: number;
  };
}

interface DecisionOutput {
  action: string;
  status: 'approved' | 'rejected' | 'deferred' | 'escalated';
  utility: number;
  confidence: number;
  causalEffect?: CausalResult;
  verificationStatus: {
    status: string;
    confidence: number;
  };
  auditHash: string;
}

// 主入口
declare function objectiveReasoning(
  input: DecisionInput,
  context: DecisionContext
): Promise<DecisionOutput>;

// 因果推論
declare function applyDoCalculus(
  graph: CausalGraph,
  intervention: Record<string, any>
): CausalResult;

declare function computeCounterfactual(
  graph: CausalGraph,
  treatment: string,
  outcome: any,
  alternative: string
): Promise<any>;

// 溯因推理
declare function abductiveInference(
  observation: Record<string, any>,
  graph: CausalGraph
): Promise<any>;
```

---

## §5. 依賴與約束

### §5.1 模組依賴

| 依賴模組 | 描述 | 引用 |
| --- | --- | --- |
| CONSTRAINTS.md | 社會權限層級定義 | 權限校驗 |
| FORMAL_VERIFIER.md | 形式化驗證引擎 | 決策路徑驗證 |
| KNOWNLEDGE_BASE.md | 資訊位元帳本 | 事實查詢 |
| CAUSAL_GRAPHS/ | 因果圖儲存庫 | 圖檢索 |
| AUDIT_TRAIL.md | 審計軌跡 | 決策記錄 |
| SANDBOX/ | 影子模擬專區 | 高風險預演 |

### §5.2 約束條件

| 約束類型 | 描述 | 邊界 |
| --- | --- | :--- |
| 認知資源 | 決策深度 × 廣度 ≤ 可用資源 | $D \times B \leq R_{cognitive}$ |
| 因果圖 | 必須為 DAG | 無循環 |
| 形式驗證 | SA-L2+ 需 FV-L3+ | confidence ≥ 0.80 |
| 影子模擬 | SA-L3+ 強制執行 | 風險閾值 ≥ 3 |
| 審計 | 高風險決策必記 | 衝突時必記 |

---

## §6. 版本與演進

| 版本 | 日期 | 變更摘要 |
| --- | :--- | --- |
| v2.2 | 2026-03 | 初始版本，對應 NoieLogicAGENTS.md §5.1 與 §5.2 |

**演化約束：** 本模組的修改不得違反 NoieLogicAGENTS.md 的不可變核心公理。任何演化提議必須記錄至 EVOLUTION_LOG.md。

---

*Logic-Engine v2.2 — 客觀推論引擎與因果推論框架*
*基於 Pearl's do-calculus、三層因果推論與因果決策理論*
*實現 NoieLogicAGENTS 的核心推理能力*
