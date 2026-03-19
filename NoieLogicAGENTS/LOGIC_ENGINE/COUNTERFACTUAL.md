# COUNTERFACTUAL.md

## 反事實推論框架 (Counterfactual Reasoning Framework v2.2)

**定義：** 本模組是 NoieLogicAGENTS 邏輯引擎的核心子模組，實現反事實推論（Counterfactual Reasoning）框架，專責處理第三層因果推論（反事實）與雙世界模型計算。

**系統定位：** 作為 LOGIC_ENGINE.md 的 L3 細節模組，專責處理因果推論的最高層級——反事實推理，回答「若當初做了不同選擇，結果會如何？」的問題。

**依賴模組：** LOGIC_ENGINE.md、CAUSAL_INFERENCE.md、ABDUCTIVE_REASONING.md、CONSTRAINTS.md

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

## §1. 反事實推理理論基礎

### §1.1 反事實定義

反事實推理是因果推論的最高層級，處理「若...會如何？」類型的問題。

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                    反事實推理定義                                  ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  【反事實問題】                                                     ║
║                                                                       ║
║  「若當初做了 X，而非 X'，結果 Y 會是什麼？」                      ║
║                                                                       ║
║  形式化表達：                                                       ║
║    P(Y_x = y | X = x', Y = y')                                    ║
║                                                                       ║
║  解釋：                                                             ║
║    在觀測到 X = x' 且 Y = y' 的條件下，                          ║
║    若 X 被干預為 x，Y 會等于 y 的機率                              ║
║                                                                       ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  【反事實 vs 干預】                                                ║
║                                                                       ║
║  干預 (Intervention):                                              ║
║    P(Y | do(X = x)) — 若我們現在強制做 X，結果會如何？            ║
║                                                                       ║
║  反事實 (Counterfactual):                                          ║
║    P(Y_x = y | X = x', Y = y') — 若當初做了不同選擇，             ║
║    在已知現在結果的情況下，結果會如何？                            ║
║                                                                       ║
║  關鍵區別：                                                         ║
║    - 干預是前瞻性的（未來）                                       ║
║    - 反事實是回溯性的（過去+假設）                                ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### §1.2 三層因果推論中的位置

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                    三層因果推論架構                                 ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  Layer 3 ────────── 反事實 (Counterfactual) ────────── 最高層級    ║
║  ══════════════════════════════════════════════════════              ║
║  「若當初做了不同選擇，結果會如何？」                              ║
║  工具：結構方程、雙世界模型                                        ║
║  數學：P(Y_x | X=x', Y=y')                                        ║
║  實現：本模組                                                       ║
║                                                                       ║
║        ═══════════════════════════════════════                       ║
║                                                                       ║
║  Layer 2 ────────── 干預 (Intervention) ────────── 中間層級         ║
║  「若我強制改變 X，Y 會如何變化？」                                ║
║  工具：do-calculus、截斷分解                                       ║
║  數學：P(Y | do(X=x))                                              ║
║  實現：CAUSAL_INFERENCE.md                                          ║
║                                                                       ║
║        ═══════════════════════════════════════                       ║
║                                                                       ║
║  Layer 1 ────────── 關聯 (Association) ────────── 基底層級         ║
║  「觀測到 X 發生時，Y 的機率是多少？」                             ║
║  工具：條件機率、貝葉斯推論                                        ║
║  數學：P(Y | X)                                                    ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

---

## §2. 雙世界模型

### §2.1 雙世界理論

```python
"""
雙世界模型實現
"""

from typing import Dict, List, Optional, Set, Any
from dataclasses import dataclass
from enum import Enum
import copy

@dataclass
class StructuralEquationModel:
    """
    結構方程模型
    
    M = (U, V, F)
    - U: 外生變數集合
    - V: 內生變數集合
    - F: 結構方程集合
    """
    exogenous_vars: Set[str]      # U: 外部變數
    endogenous_vars: Set[str]     # V: 內部變數
    structural_equations: Dict[str, str]  # F: 結構方程
    
    def evaluate(self, assignments: Dict[str, Any]) -> Dict[str, Any]:
        """
        評估結構方程
        
        根據當前賦值計算所有內生變數的值
        """
        # 拓撲排序確保父節點先計算
        evaluated = dict(assignments)
        
        for var in self._topological_order():
            if var in self.endogenous_vars:
                equation = self.structural_equations.get(var, "0")
                # 簡化實現：框架層面
                evaluated[var] = self._evaluate_equation(
                    equation, evaluated
                )
        
        return evaluated
    
    def _topological_order(self) -> List[str]:
        """獲取拓撲排序順序"""
        # 簡化實現
        return list(self.endogenous_vars)
    
    def _evaluate_equation(self, equation: str, context: Dict) -> Any:
        """評估單個結構方程"""
        # 框架實現
        return 0.0


@dataclass
class TwinWorld:
    """
    雙世界之一：事實世界或反事實世界
    
    每個世界包含：
    - 結構方程模型
    - 外生變數的賦值（固定）
    - 干預集合
    """
    sem: StructuralEquationModel
    exogenous_assignments: Dict[str, Any]  # U 值
    interventions: Dict[str, Any]  # do(X=x) 干預
    world_id: str  # "factual" 或 "counterfactual"
    
    def evaluate(self) -> Dict[str, Any]:
        """
        評估整個世界
        
        應用干預並計算所有內生變數
        """
        # 合併干預到外生賦值
        context = dict(self.exogenous_assignments)
        context.update(self.interventions)
        
        return self.sem.evaluate(context)
    
    def get(self, variable: str) -> Any:
        """獲取特定變數的值"""
        result = self.evaluate()
        return result.get(variable)


class TwinWorldModel:
    """
    雙世界模型
    
    包含：
    - 事實世界 (factual world)：實際發生的歷史
    - 反事實世界 (counterfactual world)：假設歷史
    """
    
    def __init__(self, sem: StructuralEquationModel):
        self.sem = sem
        self.factual_world: Optional[TwinWorld] = None
        self.counterfactual_world: Optional[TwinWorld] = None
    
    def setup_factual_world(
        self,
        observed_assignments: Dict[str, Any]
    ):
        """
        設定事實世界
        
        從觀測結果推斷外生變數 U 的值
        """
        # 反推外生變數：根據觀測結果推斷 U
        inferred_u = self._infer_exogenous(observed_assignments)
        
        self.factual_world = TwinWorld(
            sem=self.sem,
            exogenous_assignments=inferred_u,
            interventions={},
            world_id="factual"
        )
    
    def setup_counterfactual_world(
        self,
        treatment: str,
        treatment_value: Any
    ):
        """
        設定反事實世界
        
        設定干預 do(X=x)
        """
        # 使用與事實世界相同的外生賦值
        if self.factual_world is None:
            raise ValueError("Must setup factual world first")
        
        self.counterfactual_world = TwinWorld(
            sem=self.sem,
            exogenous_assignments=self.factual_world.exogenous_assignments,
            interventions={treatment: treatment_value},
            world_id="counterfactual"
        )
    
    def _infer_exogenous(
        self,
        observations: Dict[str, Any]
    ) -> Dict[str, Any]:
        """
        從觀測結果推斷外生變數
        
        這是反事實推理的關鍵步驟
        """
        # 框架實現：需要完整的前向模型
        inferred = {}
        
        # 簡化：假設外生變數可觀測或可推斷
        for var in self.sem.exogenous_vars:
            if var in observations:
                inferred[var] = observations[var]
        
        return inferred
```

### §2.2 ctf-calculus 與反事實推理整合

**ctf-calculus** (Correa & Bareinboim, 2025) 將 Do-Calculus 推廣至反事實領域，實現了干預與反事實的統一推理框架。本模組整合 ctf-calculus 以增強反事實計算能力。

```text
╔═══════════════════════════════════════════════════════════════════════╗
║          ctf-calculus: 反事實微積分與雙世界模型整合                ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  【ctf-calculus 核心思想】                                         ║
║                                                                       ║
║  傳統雙世界模型需要完整的結構方程模型 (SEM) 來計算反事實，         ║
║  ctf-calculus 提供了一種無需完整 SEM 即可進行反事實推理的方法。   ║
║                                                                       ║
║  【統一框架】                                                       ║
║                                                                       ║
║  ctf-calculus 的四條規則允許我們：                                 ║
║                                                                       ║
║  • 在觀測數據基礎上識別反事實機率                                   ║
║  • 減少對強假設（如確定性函數）的依賴                             ║
║  • 實現部分識別與敏感性分析                                        ║
║                                                                       ║
║  【與雙世界模型整合】                                               ║
║                                                                       ║
║  1. 識別階段：使用 ctf-calculus 規則識別可識別的反事實            ║
║  2. 計算階段：對於可識別情況，使用雙世界模型精確計算               ║
║  3. 估計階段：對於部分識別，返回可信區間                           ║
║                                                                       ║
║  【優勢對比】                                                       ║
║                                                                       ║
║  ┌─────────────────┬───────────────────┬───────────────────────┐    ║
║  │     方法        │   雙世界模型       │    ctf-calculus       │    ║
║  ├─────────────────┼───────────────────┼───────────────────────┤    ║
║  │  SEM 依賴      │      完整依賴       │     部分依賴          │    ║
║  │  識別能力      │      精確識別        │  精確+部分識別        │    ║
║  │  計算成本      │       高            │       可變            │    ║
║  │  適用場景     │  完全已知結構        │  未知/部分結構        │    ║
║  └─────────────────┴───────────────────┴───────────────────────┘    ║
║                                                                       ║
║  【引用】                                                           ║
║  Correa, J.D., & Bareinboim, E. (2025).                           ║
║  "A Calculus for Counterfactual Reasoning."                        ║
║  ICML 2025.                                                         ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

```python
"""
ctf-calculus 整合實現
增強反事實推理引擎
"""

class CTFCalculusCounterfactualEngine:
    """
    整合 ctf-calculus 的反事實引擎
    
    結合雙世界模型與 ctf-calculus 的優勢
    """
    
    def __init__(self, sem: StructuralEquationModel, causal_graph: CausalGraph):
        self.sem = sem
        self.graph = causal_graph
        self.twin_world_model = TwinWorldModel(sem)
    
    def compute_counterfactual_enhanced(
        self,
        treatment: str,
        treatment_factual: Any,
        treatment_counterfactual: Any,
        outcome: str,
        observed_outcome: Any,
        observations: Dict[str, Any]
    ) -> Dict[str, Any]:
        """
        增強反事實計算
        
        優先嘗試 ctf-calculus 識別，若失敗則使用雙世界模型
        """
        
        # ═══════════════════════════════════════════════════════════
        # STEP 1: 嘗試 ctf-calculus 識別
        # ═══════════════════════════════════════════════════════════
        
        ctf_result = self._try_ctf_identification(
            treatment, treatment_counterfactual,
            outcome, observed_outcome, observations
        )
        
        if ctf_result["identifiable"]:
            return {
                "method": "ctf-calculus",
                "counterfactual_value": ctf_result["value"],
                "probability": ctf_result["probability"],
                "confidence": ctf_result["confidence"],
                "identification_method": "ctf_rules"
            }
        
        # ═══════════════════════════════════════════════════════════
        # STEP 2: 回退至雙世界模型
        # ═══════════════════════════════════════════════════════════
        
        twin_result = self._compute_twin_world_counterfactual(
            treatment, treatment_counterfactual,
            outcome, observed_outcome, observations
        )
        
        return {
            "method": "twin_world",
            "counterfactual_value": twin_result["counterfactual_value"],
            "probability": twin_result["probability"],
            "confidence": twin_result["confidence"],
            "identification_method": "sem_evaluation"
        }
    
    def _try_ctf_identification(
        self,
        treatment: str,
        treatment_counterfactual: Any,
        outcome: str,
        observed_outcome: Any,
        observations: Dict[str, Any]
    ) -> Dict[str, Any]:
        """
        嘗試使用 ctf-calculus 識別反事實
        
        返回：(是否可識別, 識別值, 信心度)
        """
        # 簡化實現：框架層面
        
        # 檢查 CF1 條件
        # 若 (Y ⊥⊥ Z | X, W) 在 G_overline{X} 中成立
        # 可以簡化計算
        
        # 這裡需要完整的條件獨立性檢驗
        
        return {
            "identifiable": False,
            "value": None,
            "probability": None,
            "confidence": 0.0
        }
    
    def _compute_twin_world_counterfactual(
        self,
        treatment: str,
        treatment_counterfactual: Any,
        outcome: str,
        observed_outcome: Any,
        observations: Dict[str, Any]
    ) -> Dict[str, Any]:
        """
        使用雙世界模型計算反事實
        """
        # 設定事實世界
        self.twin_world_model.setup_factual_world(observations)
        
        # 設定反事實世界
        self.twin_world_model.setup_counterfactual_world(
            treatment=treatment,
            treatment_value=treatment_counterfactual
        )
        
        # 計算反事實結果
        counterfactual_value = (
            self.twin_world_model.counterfactual_world.get(outcome)
        )
        
        return {
            "counterfactual_value": counterfactual_value,
            "probability": 1.0,
            "confidence": 0.7
        }
    
    def compute_partial_identification(
        self,
        treatment: str,
        treatment_counterfactual: Any,
        outcome: str,
        observed_outcome: Any,
        observations: Dict[str, Any]
    ) -> Dict[str, Any]:
        """
        計算部分識別邊界
        
        當無法精確識別時，返回可信區間
        """
        # 使用 ctf-calculus 的部分識別理論
        
        # 獲取識別所需的混淆變數
        confounders = self._identify_confounders(treatment, outcome)
        
        # 計算邊界
        lower_bound = self._compute_lower_bound(
            treatment, treatment_counterfactual, outcome, confounders
        )
        upper_bound = self._compute_upper_bound(
            treatment, treatment_counterfactual, outcome, confounders
        )
        
        return {
            "identifiable": False,
            "partial": True,
            "lower_bound": lower_bound,
            "upper_bound": upper_bound,
            "confidence": 0.5,
            "method": "partial_identification"
        }
    
    def _identify_confounders(
        self,
        treatment: str,
        outcome: str
    ) -> List[str]:
        """識別混淆變數"""
        # 使用後門準則
        # 框架實現
        return []
    
    def _compute_lower_bound(
        self,
        treatment: str,
        treatment_value: Any,
        outcome: str,
        confounders: List[str]
    ) -> float:
        """計算下界"""
        return 0.0
    
    def _compute_upper_bound(
        self,
        treatment: str,
        treatment_value: Any,
        outcome: str,
        confounders: List[str]
    ) -> float:
        """計算上界"""
        return 1.0
```

---

## §3. 反事實計算

### §3.1 反事實計算流程

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                    反事實計算流程                                   ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  步驟 1：建構事實世界                                               ║
║  ───────────────────────────────                                    ║
║  - 根據觀測到的 (X=x', Y=y') 建構結構方程模型                      ║
║  - 反推外生變數 U 的值                                              ║
║                                                                       ║
║  步驟 2：設定反事實干預                                            ║
║  ───────────────────────────────                                    ║
║  - 在相同結構方程下，施加干預 do(X=x)                              ║
║  - 保持 U 值不變（與事實世界相同）                                 ║
║                                                                       ║
║  步驟 3：計算反事實結果                                             ║
║  ───────────────────────────────                                    ║
║  - 在反事實世界中求解 Y_x                                           ║
║  - 獲得反事實結果                                                   ║
║                                                                       ║
║  步驟 4：計算反事實機率                                            ║
║  ───────────────────────────────                                    ║
║  - 若無法精確計算，估計 P(Y_x = y | X=x', Y=y')                   ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### §3.2 反事實計算實現

```python
"""
反事實計算實現
"""

@dataclass
class CounterfactualResult:
    """反事實計算結果"""
    factual_value: Any           # 事實結果 Y(x')
    counterfactual_value: Any    # 反事實結果 Y(x)
    probability: float          # 反事實機率 P(Y_x = y | X=x', Y=y')
    causal_effect: float        # 個體處理效應 (ITE)
    assumptions: List[str]      # 識別假設
    confidence: float           # 信心度


class CounterfactualEngine:
    """
    反事實計算引擎
    """
    
    def __init__(self, sem: StructuralEquationModel):
        self.sem = sem
        self.twin_world_model = TwinWorldModel(sem)
    
    def compute_counterfactual(
        self,
        treatment: str,
        treatment_factual: Any,      # 實際發生的處理
        treatment_counterfactual: Any, # 假設的處理
        outcome: str,
        observed_outcome: Any,
        observations: Dict[str, Any]
    ) -> CounterfactualResult:
        """
        計算反事實結果
        
        問題：「若當初選擇 treatment_counterfactual，結果會如何？」
        """
        
        # ═══════════════════════════════════════════════════
        # STEP 1: 建構事實世界
        # ═══════════════════════════════════════════════════
        
        self.twin_world_model.setup_factual_world(observations)
        
        factual_value = self.twin_world_model.factual_world.get(outcome)
        
        # ═══════════════════════════════════════════════════
        # STEP 2: 設定反事實干預
        # ═══════════════════════════════════════════════════
        
        self.twin_world_model.setup_counterfactual_world(
            treatment=treatment,
            treatment_value=treatment_counterfactual
        )
        
        # ═══════════════════════════════════════════════════
        # STEP 3: 計算反事實結果
        # ═══════════════════════════════════════════════════
        
        counterfactual_value = (
            self.twin_world_model.counterfactual_world.get(outcome)
        )
        
        # ═══════════════════════════════════════════════════
        # STEP 4: 計算個體處理效應
        # ═══════════════════════════════════════════════════
        
        ite = counterfactual_value - factual_value if isinstance(
            factual_value, (int, float)
        ) else 0.0
        
        # ═══════════════════════════════════════════════════
        # STEP 5: 估計反事實機率
        # ═══════════════════════════════════════════════════
        
        probability = self._estimate_counterfactual_probability(
            treatment,
            treatment_factual,
            treatment_counterfactual,
            outcome,
            observed_outcome
        )
        
        return CounterfactualResult(
            factual_value=factual_value,
            counterfactual_value=counterfactual_value,
            probability=probability,
            causal_effect=ite,
            assumptions=[
                "consistency",
                "conditional_exogeneity",
                "no_measurement_error"
            ],
            confidence=0.8
        )
    
    def _estimate_counterfactual_probability(
        self,
        treatment: str,
        treatment_factual: Any,
        treatment_counterfactual: Any,
        outcome: str,
        observed_outcome: Any
    ) -> float:
        """
        估計反事實機率
        
        P(Y_x = y | X=x', Y=y')
        
        方法：
        1. 若結構方程已知，精確計算
        2. 若需估計，使用敏感性分析
        """
        # 框架實現
        # 實際實現需要數據支持
        
        # 簡化：返回確定性估計
        return 1.0
    
    def compute_ite_distribution(
        self,
        treatment: str,
        outcome: str,
        individual_covariates: Dict[str, Any]
    ) -> List[float]:
        """
        計算個體處理效應分佈
        
        用於異質性分析
        """
        ite_values = []
        
        # 對每個個體計算 ITE
        # 框架實現
        
        return ite_values
```

---

## §4. 反事實因果推論

### §4.1 反事實因果推論類型

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                    反事實因果推論類型                                 ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  1. 責任歸因 (Attribution)                                          ║
║     ───────────────────────────────                                  ║
║     「誰該為結果負責？」                                             ║
║     比較：事實結果 vs 反事實結果                                     ║
║                                                                       ║
║  2. 代理因果推論 (Proxy Causation)                                   ║
║     ───────────────────────────────                                  ║
║     「若 X 沒有發生，Y 會發生嗎？」                                 ║
║     使用反事實定義因果                                               ║
║                                                                       ║
║  3. 結果解釋 (Outcome Explanation)                                  ║
║     ───────────────────────────────                                  ║
║     「為什麼結果是 Y？」                                            ║
║     生成導致結果的因果鏈                                            ║
║                                                                       ║
║  4. 期望損失估計 (Expected Loss Estimation)                         ║
║     ───────────────────────────────                                  ║
║     「選擇 A 而非 B 的預期損失是多少？」                             ║
║     使用反事實計算機會成本                                           ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### §4.2 反事實因果推論實現

```python
"""
反事實因果推論實現
"""

@dataclass
class AttributionResult:
    """責任歸因結果"""
    responsible_variable: str
    attribution_score: float
    counterfactual_comparison: Dict[str, Any]


@dataclass
class ExplanationResult:
    """結果解釋"""
    outcome: str
    causal_chain: List[str]
    explanation_confidence: float


class CounterfactualCausalInference:
    """
    反事實因果推論器
    """
    
    def __init__(self, causal_graph: "CausalGraph"):
        self.causal_graph = causal_graph
    
    def attribute_responsibility(
        self,
        outcome: str,
        observed_value: Any,
        potential_causes: List[str],
        observations: Dict[str, Any]
    ) -> AttributionResult:
        """
        責任歸因
        
        識別導致觀測結果的主要原因
        """
        best_cause = None
        best_score = -float('inf')
        
        for cause in potential_causes:
            # 計算反事實：若該原因未發生，結果會如何？
            cf_result = self._compute_counterfactual_omission(
                cause, outcome, observations
            )
            
            # 歸因分數：反事實結果與觀測結果的差異
            attribution_score = self._compute_attribution_score(
                observed_value,
                cf_result.counterfactual_value
            )
            
            if attribution_score > best_score:
                best_score = attribution_score
                best_cause = cause
        
        return AttributionResult(
            responsible_variable=best_cause,
            attribution_score=best_score,
            counterfactual_comparison={}
        )
    
    def explain_outcome(
        self,
        outcome: str,
        outcome_value: Any,
        observations: Dict[str, Any]
    ) -> ExplanationResult:
        """
        結果解釋
        
        生成導致結果的因果鏈解釋
        """
        # 從結果回溯到根因
        causal_chain = self._backtrack_causal_chain(
            outcome, observations
        )
        
        return ExplanationResult(
            outcome=outcome_value,
            causal_chain=causal_chain,
            explanation_confidence=0.7
        )
    
    def estimate_expected_loss(
        self,
        action_taken: str,
        action_alternative: str,
        outcome: str,
        utility_function: Dict[Any, float],
        observations: Dict[str, Any]
    ) -> float:
        """
        估計期望損失
        
        計算選擇 action_taken 而非 action_alternative 的機會成本
        """
        # 計算事實結果的效用
        factual_result = self._compute_factual_outcome(
            action_taken, outcome, observations
        )
        factual_utility = utility_function.get(factual_result, 0.0)
        
        # 計算反事實結果的效用
        cf_result = self._compute_counterfactual_switch(
            action_taken, action_alternative, outcome, observations
        )
        cf_utility = utility_function.get(cf_result.counterfactual_value, 0.0)
        
        # 期望損失
        expected_loss = factual_utility - cf_utility
        
        return expected_loss
    
    def _compute_counterfactual_omission(
        self,
        variable: str,
        outcome: str,
        observations: Dict[str, Any]
    ) -> CounterfactualResult:
        """
        計算省略反事實
        
        「若該變數未發生，結果會如何？」
        """
        # 框架實現
        return CounterfactualResult(
            factual_value=0.0,
            counterfactual_value=0.0,
            probability=0.5,
            causal_effect=0.0,
            assumptions=[],
            confidence=0.5
        )
    
    def _compute_attribution_score(
        self,
        observed: Any,
        counterfactual: Any
    ) -> float:
        """計算歸因分數"""
        if isinstance(observed, (int, float)) and isinstance(
            counterfactual, (int, float)
        ):
            return abs(observed - counterfactual)
        return 0.0
    
    def _backtrack_causal_chain(
        self,
        outcome: str,
        observations: Dict[str, Any]
    ) -> List[str]:
        """回溯因果鏈"""
        chain = [outcome]
        
        # 簡化實現
        current = outcome
        
        for _ in range(5):  # 最多5步
            parents = self.causal_graph.get_parents(current)
            if not parents:
                break
            
            # 選擇最相關的父節點
            # 框架實現
            break
        
        return chain
    
    def _compute_factual_outcome(
        self,
        action: str,
        outcome: str,
        observations: Dict[str, Any]
    ) -> Any:
        """計算事實結果"""
        return observations.get(outcome, 0.0)
    
    def _compute_counterfactual_switch(
        self,
        action_taken: str,
        action_alternative: str,
        outcome: str,
        observations: Dict[str, Any]
    ) -> CounterfactualResult:
        """計算替換反事實"""
        # 框架實現
        return CounterfactualResult(
            factual_value=0.0,
            counterfactual_value=0.0,
            probability=0.5,
            causal_effect=0.0,
            assumptions=[],
            confidence=0.5
        )
```

---

## §5. 反事實決策應用

### §5.1 反事實輔助決策

```python
"""
反事實輔助決策
"""

class CounterfactualDecisionSupport:
    """
    反事實決策支持系統
    
    使用反事實推理輔助高風險決策
    """
    
    def __init__(self, causal_graph: "CausalGraph", sem: StructuralEquationModel):
        self.causal_graph = causal_graph
        self.sem = sem
        self.counterfactual_engine = CounterfactualEngine(sem)
        self.causal_inference = CounterfactualCausalInference(causal_graph)
    
    def analyze_decision(
        self,
        decision: str,
        outcome: str,
        alternatives: List[str],
        observations: Dict[str, Any],
        utility_function: Dict[Any, float]
    ) -> Dict[str, Any]:
        """
        分析決策
        
        使用反事實推理評估決策品質
        """
        results = {
            "decision": decision,
            "analysis": {}
        }
        
        for alt in alternatives:
            if alt == decision:
                continue
            
            # 計算期望損失
            expected_loss = self.causal_inference.estimate_expected_loss(
                decision, alt, outcome, utility_function, observations
            )
            
            # 計算責任歸因
            attribution = self.causal_inference.attribute_responsibility(
                outcome, observations.get(outcome, 0.0),
                [decision, alt], observations
            )
            
            results["analysis"][alt] = {
                "expected_loss": expected_loss,
                "attribution": attribution.attribution_score,
                "recommendation": "avoid" if expected_loss > 0 else "acceptable"
            }
        
        return results
    
    def what_if_analysis(
        self,
        intervention: Dict[str, Any],
        outcome: str,
        baseline_observations: Dict[str, Any]
    ) -> Dict[str, Any]:
        """
        假設分析
        
        「若我們干預 X，結果會如何？」
        """
        treatment = list(intervention.keys())[0]
        treatment_value = intervention[treatment]
        
        # 使用反事實引擎計算
        result = self.counterfactual_engine.compute_counterfactual(
            treatment=treatment,
            treatment_factual=baseline_observations.get(treatment, 0),
            treatment_counterfactual=treatment_value,
            outcome=outcome,
            observed_outcome=baseline_observations.get(outcome, 0),
            observations=baseline_observations
        )
        
        return {
            "intervention": intervention,
            "factual": result.factual_value,
            "counterfactual": result.counterfactual_value,
            "causal_effect": result.causal_effect,
            "confidence": result.confidence
        }
```

---

## §6. 與 LOGIC_ENGINE 整合

### §6.1 整合接口

```python
"""
反事實推理與邏輯引擎整合
"""

def apply_counterfactual_to_decision(
    decision_input: Dict,
    context: "DecisionContext"
) -> "DecisionOutput":
    """
    將反事實推理應用於決策流程
    
    當需要回答「若...會如何？」類型問題時觸發
    """
    from CAUSAL_INFERENCE import CausalInferenceEngine
    
    # 1. 提取決策相關變數
    decision_var = decision_input.get("decision")
    outcome_var = decision_input.get("outcome")
    alternatives = decision_input.get("alternatives", [])
    
    # 2. 獲取因果圖
    graph = BuildOrRetrieveCausalGraph(decision_input, context)
    
    # 3. 初始化反事實引擎
    # 需要結構方程模型
    sem = StructuralEquationModel(
        exogenous_vars=set(),
        endogenous_vars=set(),
        structural_equations={}
    )
    
    cf_engine = CounterfactualEngine(sem)
    cf_inference = CounterfactualCausalInference(graph)
    
    # 4. 執行反事實分析
    results = {}
    
    for alt in alternatives:
        cf_result = cf_engine.compute_counterfactual(
            treatment=decision_var,
            treatment_factual=decision_input.get("current_value", ""),
            treatment_counterfactual=alt,
            outcome=outcome_var,
            observed_outcome=decision_input.get("observed_outcome", 0),
            observations=decision_input.get("observations", {})
        )
        
        results[alt] = {
            "counterfactual_value": cf_result.counterfactual_value,
            "causal_effect": cf_result.causal_effect,
            "confidence": cf_result.confidence
        }
    
    # 5. 生成決策建議
    best_alternative = max(
        results.keys(),
        key=lambda a: results[a]["causal_effect"]
    )
    
    return {
        "recommended_action": best_alternative,
        "analysis": results,
        "method": "counterfactual_reasoning"
    }
```

---

## §7. 依賴與約束

### §7.1 模組依賴

| 依賴模組 | 描述 | 引用 |
| --- | --- | --- |
| LOGIC_ENGINE.md | 父模組 | 調用入口 |
| CAUSAL_INFERENCE.md | 因果推論引擎 | do-calculus |
| ABDUCTIVE_REASONING.md | 溯因推理 | 異常解釋 |
| CONSTRAINTS.md | 社會權限層級 | SA-L 權限校驗 |
| FORMAL_VERIFIER.md | 形式化驗證 | 邏輯閉包驗證 |

### §7.2 約束條件

| 約束類型 | 描述 | 邊界 |
| --- | --- | :--- |
| 結構方程 | 需要完整的 SEM | 框架實現 |
| 外生變數 | 需可推斷或觀測 | 有限制 |
| 信心度閾值 | 信心度 < 0.3 標記不確定 | confidence ≥ 0.3 |
| 反事實深度 | 最大回溯深度 | ≤ 5 步 |

---

## §8. 版本與演進

| 版本 | 日期 | 變更摘要 |
| --- | :--- | :--- |
| v2.2 | 2026-03 | 初始版本，對應 LOGIC_ENGINE.md §2.3 |
| v2.3 | 2026-03 | 整合 ctf-calculus 增強反事實推理 |

**演化約束：** 本模組的修改不得違反 NoieLogicAGENTS.md 的不可變核心公理。任何演化提議必須記錄至 EVOLUTION_LOG.md。

---

*Counterfactual Reasoning Framework v2.2 — 雙世界模型與反事實因果推論*
*實現第三層因果推論的核心能力*
