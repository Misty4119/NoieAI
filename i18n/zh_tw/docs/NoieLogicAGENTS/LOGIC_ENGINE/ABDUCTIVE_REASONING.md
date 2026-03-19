# ABDUCTIVE_REASONING.md

## 溯因推理模組 (Abductive Reasoning Module v2.2)

**定義：** 本模組是 NoieLogicAGENTS 邏輯引擎的核心子模組，實現溯因推理（Abductive Reasoning）框架，專責處理異常檢測、候選原因生成與最短解釋選擇。

**系統定位：** 作為 LOGIC_ENGINE.md 的 L3 細節模組，專責處理第三層認知能力——當觀測結果無法被現有因果圖解釋時，生成新的因果假設。

**依賴模組：** LOGIC_ENGINE.md、CAUSAL_INFERENCE.md、CONSTRAINTS.md、FORMAL_VERIFIER.md

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

## §1. 溯因推理理論基礎

### §1.1 溯因推理定義

溯因推理（Abduction）是由 Charles Sanders Peirce 提出的第三種推理形式，與演繹（Deduction）和歸納（Induction）並列。

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                    三種推理形式比較                                 ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  【演繹推理 (Deduction)】                                            ║
║  ─────────────────────────────────────                              ║
║  前提：所有 A 是 B，x 是 A                                          ║
║  結論：x 是 B                                                       ║
║  性質：保真推理（若前提為真，結論必然為真）                         ║
║                                                                       ║
║  【歸納推理 (Induction)】                                            ║
║  ─────────────────────────────────────                              ║
║  前提：x₁ 是 A 是 B，x₂ 是 A 是 B，...                            ║
║  結論：所有 A 是 B                                                  ║
║  性質：概率推理（結論為概然）                                       ║
║                                                                       ║
║  【溯因推理 (Abduction)】                                            ║
║  ─────────────────────────────────────                              ║
║  前提：觀測到 B，規則 A → B                                         ║
║  結論：x 可能是 A                                                   ║
║  性質：最優解釋推理（結論為假設）                                  ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### §1.2 溯因推理在因果框架中的角色

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                    三層因果推論與溯因推理                           ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  Layer 3 ────────── 反事實 (Counterfactual) ────────── 最高層級    ║
║  「若當初做了不同選擇，結果會如何？」                               ║
║                                                                       ║
║        ═══════════════════════════════════════                       ║
║                                                                       ║
║  Layer 2 ────────── 干預 (Intervention) ────────── 中間層級         ║
║  「若我強制改變 X，Y 會如何變化？」                                 ║
║                                                                       ║
║        ═══════════════════════════════════════                       ║
║                                                                       ║
║  Layer 1 ────────── 關聯 (Association) ────────── 基底層級          ║
║  「觀測到 X 發生時，Y 的機率是多少？」                             ║
║                                                                       ║
║        ═══════════════════════════════════════                       ║
║                                                                       ║
║  【溯因推理位置】                                                    ║
║  ─────────────────────────────────────                              ║
║  溯因推理是「因果學習」的核心：                                     ║
║  當觀測與因果圖預測不一致時，生成新的因果假設                       ║
║  這是從數據中發現因果結構的關鍵能力                                 ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

---

## §2. 異常檢測

### §2.1 異常定義與類型

```python
"""
異常檢測模組
"""

from typing import Dict, List, Optional, Set, Tuple
from dataclasses import dataclass
from enum import Enum
import math

class AnomalyType(Enum):
    """異常類型"""
    RESIDUAL = "residual"           # 殘差異常
    STRUCTURAL = "structural"        # 結構異常
    DISTRIBUTIONAL = "distributional" # 分佈異常
    TEMPORAL = "temporal"            # 時序異常
    CAUSAL = "causal"               # 因果異常

@dataclass
class Anomaly:
    """異常結構"""
    anomaly_type: AnomalyType
    observed_value: float
    predicted_value: float
    residual: float
    severity: float  # 0-1
    timestamp: Optional[str] = None
    context: Optional[Dict] = None

@dataclass
class AnomalyDetectionResult:
    """異常檢測結果"""
    is_anomaly: bool
    anomaly: Optional[Anomaly]
    detection_method: str
    confidence: float
    threshold_used: float


class AnomalyDetector:
    """
    異常檢測器
    """
    
    def __init__(
        self,
        residual_threshold: float = 2.0,
        severity_threshold: float = 0.5
    ):
        self.residual_threshold = residual_threshold
        self.severity_threshold = severity_threshold
    
    def detect_residual_anomaly(
        self,
        observed: float,
        predicted: float,
        std_error: float = 1.0
    ) -> AnomalyDetectionResult:
        """
        檢測殘差異常
        
        使用標準化殘差：
        z = (observed - predicted) / std_error
        
        若 |z| > threshold，則為異常
        """
        residual = observed - predicted
        z_score = abs(residual / std_error) if std_error > 0 else float('inf')
        
        is_anomaly = z_score > self.residual_threshold
        
        # 計算嚴重度
        severity = min(z_score / self.residual_threshold, 1.0)
        
        anomaly = None
        if is_anomaly:
            anomaly = Anomaly(
                anomaly_type=AnomalyType.RESIDUAL,
                observed_value=observed,
                predicted_value=predicted,
                residual=residual,
                severity=severity
            )
        
        return AnomalyDetectionResult(
            is_anomaly=is_anomaly,
            anomaly=anomaly,
            detection_method="residual_zscore",
            confidence=min(severity, 0.95),
            threshold_used=self.residual_threshold
        )
    
    def detect_distributional_anomaly(
        self,
        value: float,
        distribution_params: Dict
    ) -> AnomalyDetectionResult:
        """
        檢測分佈異常
        
        使用分佈假設（如正態分佈）計算尾部機率
        """
        # 簡化實現：使用 Z-score
        mean = distribution_params.get("mean", 0)
        std = distribution_params.get("std", 1)
        
        return self.detect_residual_anomaly(value, mean, std)
    
    def detect_temporal_anomaly(
        self,
        time_series: List[float],
        window_size: int = 10
    ) -> List[AnomalyDetectionResult]:
        """
        檢測時序異常
        
        使用滑動窗口檢測偏離趨勢的點
        """
        anomalies = []
        
        for i in range(window_size, len(time_series)):
            # 計算窗口均值
            window = time_series[i-window_size:i]
            predicted = sum(window) / window_size
            
            # 檢測異常
            result = self.detect_residual_anomaly(
                time_series[i],
                predicted
            )
            
            if result.is_anomaly and result.anomaly:
                result.anomaly.timestamp = f"t={i}"
            
            anomalies.append(result)
        
        return anomalies
```

### §2.2 因果異常檢測

```python
"""
因果異常檢測
"""

class CausalAnomalyDetector:
    """
    因果異常檢測器
    
    檢測觀測結果是否偏離因果圖的預測
    """
    
    def __init__(self, causal_engine: "CausalInferenceEngine"):
        self.causal_engine = causal_engine
        self.anomaly_detector = AnomalyDetector()
    
    def detect_causal_anomaly(
        self,
        graph: "CausalGraph",
        observation: Dict[str, float],
        conditions: Dict[str, float]
    ) -> AnomalyDetectionResult:
        """
        檢測因果異常
        
        步驟：
        1. 使用因果圖預測結果
        2. 比較預測與觀測
        3. 若偏離超過閾值，標記為異常
        """
        # 從觀測中提取目標變數
        target_var = list(observation.keys())[0]
        
        # 檢查是否為圖中的節點
        if target_var not in graph.nodes:
            return AnomalyDetectionResult(
                is_anomaly=False,
                anomaly=None,
                detection_method="causal_not_applicable",
                confidence=0.0,
                threshold_used=0.0
            )
        
        # 使用因果引擎預測
        treatment = list(conditions.keys())[0] if conditions else None
        
        if treatment:
            effect_result = self.causal_engine.estimate_causal_effect(
                X=treatment,
                Y=target_var
            )
            predicted = effect_result.ate
        else:
            # 無干預，使用基本預測
            predicted = 0.0
        
        observed = observation[target_var]
        
        # 檢測異常
        return self.anomaly_detector.detect_residual_anomaly(
            observed,
            predicted
        )
```

---

## §3. 候選原因生成

### §3.1 原因空間定義

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                    候選原因空間                                      ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  當觀測到異常時，搜尋候選原因的空間定義：                           ║
║                                                                       ║
║  C(異常) = { c | 異常 ∈ Consequence(c) }                          ║
║                                                                       ║
║  其中：                                                              ║
║    - c 為候選原因                                                   ║
║    - Consequence(c) 為原因 c 可能導致的後果集合                     ║
║                                                                       ║
║  候選原因類型：                                                      ║
║    1. 直接原因：直接指向異常節點的邊的起點                          ║
║    2. 間接原因：通過因果路徑影響異常的節點                         ║
║    3. 根本原因：無其他原因的原因（無入邊或入邊來自外生變數）       ║
║    4. 混淆原因：通過後門路徑影響異常的節點                        ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### §3.2 候選原因搜尋演算法

```python
"""
候選原因生成
"""

@dataclass
class CandidateCause:
    """候選原因結構"""
    variable: str
    cause_type: str  # direct, indirect, root, confounding
    causal_path: List[str]  # 從原因到異常的路徑
    probability: float  # 後驗機率
    complexity: float  # 柯爾莫哥洛夫複雜度


class CandidateCauseGenerator:
    """
    候選原因生成器
    """
    
    def __init__(self, graph: "CausalGraph"):
        self.graph = graph
    
    def generate_candidates(
        self,
        anomaly_variable: str,
        max_depth: int = 3
    ) -> List[CandidateCause]:
        """
        生成候選原因列表
        """
        candidates = []
        
        # 1. 搜尋直接原因
        direct_causes = self._find_direct_causes(anomaly_variable)
        for cause in direct_causes:
            candidates.append(CandidateCause(
                variable=cause,
                cause_type="direct",
                causal_path=[cause, anomaly_variable],
                probability=0.8,
                complexity=self._estimate_complexity(cause)
            ))
        
        # 2. 搜尋間接原因
        indirect_causes = self._find_indirect_causes(
            anomaly_variable, max_depth
        )
        for cause in indirect_causes:
            path = self._find_causal_path(cause, anomaly_variable)
            candidates.append(CandidateCause(
                variable=cause,
                cause_type="indirect",
                causal_path=path,
                probability=0.5,
                complexity=self._estimate_complexity(cause)
            ))
        
        # 3. 識別根本原因
        root_causes = self._find_root_causes(anomaly_variable)
        for cause in root_causes:
            path = self._find_causal_path(cause, anomaly_variable)
            candidates.append(CandidateCause(
                variable=cause,
                cause_type="root",
                causal_path=path,
                probability=0.9,
                complexity=self._estimate_complexity(cause)
            ))
        
        # 4. 搜尋混淆原因
        confounding_causes = self._find_confounding_causes(anomaly_variable)
        for cause in confounding_causes:
            path = self._find_backdoor_path(cause, anomaly_variable)
            candidates.append(CandidateCause(
                variable=cause,
                cause_type="confounding",
                causal_path=path,
                probability=0.4,
                complexity=self._estimate_complexity(cause)
            ))
        
        return candidates
    
    def _find_direct_causes(self, target: str) -> List[str]:
        """尋找直接原因"""
        return list(self.graph.get_parents(target))
    
    def _find_indirect_causes(
        self, 
        target: str, 
        max_depth: int
    ) -> List[str]:
        """尋找間接原因"""
        indirect = []
        visited = {target}
        
        def dfs(node: str, depth: int):
            if depth >= max_depth:
                return
            
            for parent in self.graph.get_parents(node):
                if parent not in visited:
                    visited.add(parent)
                    # 排除直接原因
                    if parent not in self.graph.get_parents(target):
                        indirect.append(parent)
                    dfs(parent, depth + 1)
        
        dfs(target, 0)
        return indirect
    
    def _find_root_causes(self, target: str) -> List[str]:
        """尋找根本原因"""
        root_causes = []
        
        # 遍歷所有到 target 的路徑
        for node in self.graph.nodes:
            if node == target:
                continue
            
            # 檢查是否為根節點（無父節點或父節點為外生變數）
            parents = self.graph.get_parents(node)
            if len(parents) == 0:
                # 檢查是否有路徑到 target
                if self._has_path_to(node, target):
                    root_causes.append(node)
        
        return root_causes
    
    def _find_confounding_causes(self, target: str) -> List[str]:
        """尋找混淆原因"""
        # 混淆原因：通過後門路徑影響 target 的節點
        # 簡化實現
        return []
    
    def _has_path_to(self, source: str, target: str) -> bool:
        """檢查是否有從 source 到 target 的路徑"""
        visited = set()
        queue = [source]
        
        while queue:
            node = queue.pop(0)
            if node == target:
                return True
            
            if node in visited:
                continue
            
            visited.add(node)
            
            for child in self.graph.get_children(node):
                if child not in visited:
                    queue.append(child)
        
        return False
    
    def _find_causal_path(
        self, 
        source: str, 
        target: str
    ) -> List[str]:
        """尋找從 source 到 target 的因果路徑"""
        path = []
        visited = set()
        
        def dfs(node: str, current_path: List[str]):
            if node == target:
                path.extend(current_path + [node])
                return True
            
            visited.add(node)
            
            for child in self.graph.get_children(node):
                if child not in visited:
                    if dfs(child, current_path + [node]):
                        return True
            
            return False
        
        dfs(source, [])
        return path
    
    def _find_backdoor_path(
        self, 
        source: str, 
        target: str
    ) -> List[str]:
        """尋找後門路徑"""
        # 簡化實現
        return []
    
    def _estimate_complexity(self, variable: str) -> float:
        """
        估計變數的柯爾莫哥洛夫複雜度
        
        簡化：使用變數名的長度和圖中的連接度作為代理
        """
        name_complexity = len(variable) / 10.0
        
        connections = (
            len(self.graph.get_parents(variable)) +
            len(self.graph.get_children(variable))
        )
        structural_complexity = connections / 10.0
        
        return min(name_complexity + structural_complexity, 1.0)
```

---

## §4. 最短解釋選擇

### §4.1 柯爾莫哥洛夫複雜度

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                    最短描述優先原則                                   ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  【柯爾莫哥洛夫複雜度定義】                                          ║
║                                                                       ║
║  K(x) = 生成 x 的最短程式的長度                                     ║
║                                                                       ║
║  在因果推理中：                                                       ║
║    K(解釋) ∝ 解釋的描述長度 + 因果路徑長度                          ║
║                                                                       ║
║  【最短解釋選擇準則】                                                ║
║                                                                       ║
║  選擇 K(解釋) 最小的候選原因：                                       ║
║    Best = argmin_c K(c | 異常)                                      ║
║                                                                       ║
║  這體現了「奧卡姆剃刀」原則：                                        ║
║    「如無必要，勿增實體」                                           ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### §4.2 最優解釋選擇演算法

```python
"""
最短解釋選擇
"""

class BestExplanationSelector:
    """
    最短解釋選擇器
    
    使用柯爾莫哥洛夫複雜度選擇最優解釋
    """
    
    def __init__(self):
        self.kolmogorov_constant = 1.0  # 歸一化常數
    
    def select_best_explanation(
        self,
        candidates: List[CandidateCause],
        observation: Dict
    ) -> CandidateCause:
        """
        選擇最佳解釋
        
        選擇標準：
        P(原因|異常) ∝ P(異常|原因) × P(原因) / K(原因)
        
        其中：
        - P(異常|原因)：原因對異常的解釋力
        - P(原因)：原因的先驗機率
        - K(原因)：原因的柯爾莫哥洛夫複雜度
        """
        scored_candidates = []
        
        for candidate in candidates:
            # 計算後驗分數
            posterior = self._compute_posterior(candidate, observation)
            
            # 考慮複雜度
            complexity_penalty = candidate.complexity * self.kolmogorov_constant
            
            # 最終分數
            score = posterior - complexity_penalty
            
            scored_candidates.append((candidate, score))
        
        # 按分數排序
        scored_candidates.sort(key=lambda x: x[1], reverse=True)
        
        # 返回最高分
        return scored_candidates[0][0] if scored_candidates else None
    
    def _compute_posterior(
        self,
        candidate: CandidateCause,
        observation: Dict
    ) -> float:
        """
        計算後驗機率
        
        P(原因|異常) ∝ P(異常|原因) × P(原因)
        """
        # 先驗（基於原因類型）
        prior = self._compute_prior(candidate)
        
        # 似然（基於因果路徑）
        likelihood = self._compute_likelihood(candidate, observation)
        
        # 後驗
        posterior = prior * likelihood
        
        return posterior
    
    def _compute_prior(self, candidate: CandidateCause) -> float:
        """計算先驗機率"""
        # 根本原因最高優先級
        type_prior = {
            "root": 0.9,
            "direct": 0.7,
            "indirect": 0.4,
            "confounding": 0.3
        }
        
        return type_prior.get(candidate.cause_type, 0.5)
    
    def _compute_likelihood(
        self,
        candidate: CandidateCause,
        observation: Dict
    ) -> float:
        """
        計算似然
        
        基於因果路徑的強度
        """
        # 簡化：路徑越短，似然越高
        path_length = len(candidate.causal_path)
        
        if path_length == 0:
            return 0.0
        
        return 1.0 / path_length
    
    def rank_explanations(
        self,
        candidates: List[CandidateCause],
        observation: Dict
    ) -> List[Tuple[CandidateCause, float]]:
        """
        對所有候選原因排序
        
        返回：(原因, 分數) 的有序列表
        """
        scored = []
        
        for candidate in candidates:
            posterior = self._compute_posterior(candidate, observation)
            complexity_penalty = candidate.complexity * self.kolmogorov_constant
            score = posterior - complexity_penalty
            
            scored.append((candidate, score))
        
        return sorted(scored, key=lambda x: x[1], reverse=True)
```

---

## §5. 溯因推理流程

### §5.1 完整溯因推理流程

```python
"""
溯因推理完整流程
"""

@dataclass
class AbductiveResult:
    """溯因推理結果"""
    status: str  # "anomaly_detected", "no_anomaly", "inconclusive"
    anomaly: Optional[Anomaly]
    best_explanation: Optional[CandidateCause]
    all_candidates: List[CandidateCause]
    ranked_explanations: List[Tuple[CandidateCause, float]]
    graph_update_proposed: bool
    confidence: float


class AbductiveReasoner:
    """
    溯因推理器
    """
    
    def __init__(
        self,
        causal_engine: "CausalInferenceEngine",
        graph: "CausalGraph"
    ):
        self.causal_engine = causal_engine
        self.graph = graph
        self.anomaly_detector = AnomalyDetector()
        self.cause_generator = CandidateCauseGenerator(graph)
        self.explanation_selector = BestExplanationSelector()
    
    def infer(
        self,
        observation: Dict[str, float],
        conditions: Optional[Dict[str, float]] = None
    ) -> AbductiveResult:
        """
        執行完整溯因推理流程
        
        步驟：
        1. 異常檢測
        2. 若檢測到異常，生成候選原因
        3. 選擇最優解釋
        4. 若信心度足夠，提議因果圖更新
        """
        # ═══════════════════════════════════════════════════
        # STEP 1: 異常檢測
        # ═══════════════════════════════════════════════════
        
        target_var = list(observation.keys())[0]
        observed_value = observation[target_var]
        
        # 使用因果引擎預測
        if conditions:
            treatment = list(conditions.keys())[0]
            effect_result = self.causal_engine.estimate_causal_effect(
                X=treatment,
                Y=target_var
            )
            predicted = effect_result.ate
        else:
            predicted = 0.0
        
        anomaly_result = self.anomaly_detector.detect_residual_anomaly(
            observed_value,
            predicted
        )
        
        # ═══════════════════════════════════════════════════
        # 若無異常，返回正常
        # ═══════════════════════════════════════════════════
        
        if not anomaly_result.is_anomaly:
            return AbductiveResult(
                status="no_anomaly",
                anomaly=None,
                best_explanation=None,
                all_candidates=[],
                ranked_explanations=[],
                graph_update_proposed=False,
                confidence=anomaly_result.confidence
            )
        
        # ═══════════════════════════════════════════════════
        # STEP 2: 生成候選原因
        # ═══════════════════════════════════════════════════
        
        candidates = self.cause_generator.generate_candidates(target_var)
        
        if not candidates:
            return AbductiveResult(
                status="inconclusive",
                anomaly=anomaly_result.anomaly,
                best_explanation=None,
                all_candidates=[],
                ranked_explanations=[],
                graph_update_proposed=False,
                confidence=0.3
            )
        
        # ═══════════════════════════════════════════════════
        # STEP 3: 選擇最優解釋
        # ═══════════════════════════════════════════════════
        
        ranked = self.explanation_selector.rank_explanations(
            candidates, observation
        )
        
        best = ranked[0][0] if ranked else None
        
        # ═══════════════════════════════════════════════════
        # STEP 4: 提議因果圖更新（若信心度足夠）
        # ═══════════════════════════════════════════════════
        
        UPDATE_THRESHOLD = 0.8
        
        graph_update_proposed = False
        if best and best.probability > UPDATE_THRESHOLD:
            graph_update_proposed = True
            # 記錄至審計
            self._log_graph_update_proposal(best, observation)
        
        return AbductiveResult(
            status="anomaly_detected",
            anomaly=anomaly_result.anomaly,
            best_explanation=best,
            all_candidates=candidates,
            ranked_explanations=ranked[:5],  # 前5名
            graph_update_proposed=graph_update_proposed,
            confidence=best.probability if best else 0.3
        )
    
    def _log_graph_update_proposal(
        self,
        cause: CandidateCause,
        observation: Dict
    ):
        """記錄因果圖更新提議"""
        # 實際實現：寫入 AUDIT_TRAIL
        pass
```

---

## §6. 與 LOGIC_ENGINE 整合

### §6.1 整合接口

```python
"""
溯因推理與邏輯引擎整合
"""

def apply_abductive_reasoning_to_observation(
    observation: Dict,
    causal_graph: "CausalGraph",
    context: "DecisionContext"
) -> "AbductiveResult":
    """
    將溯因推理應用於觀測分析
    
    當決策引擎檢測到異常時觸發：
    1. 分析觀測結果
    2. 生成候選因果解釋
    3. 選擇最優解釋
    4. 提議因果圖更新
    """
    from CAUSAL_INFERENCE import CausalInferenceEngine
    
    # 初始化因果引擎
    engine = CausalInferenceEngine(causal_graph)
    
    # 執行溯因推理
    reasoner = AbductiveReasoner(engine, causal_graph)
    result = reasoner.infer(observation)
    
    # 記錄至審計
    if result.status == "anomaly_detected":
        # 記錄異常檢測
        pass
    
    return result
```

---

## §7. 依賴與約束

### §7.1 模組依賴

| 依賴模組 | 描述 | 引用 |
| --- | --- | --- |
| LOGIC_ENGINE.md | 父模組 | 調用入口 |
| CAUSAL_INFERENCE.md | 因果推論引擎 | 異常預測 |
| CONSTRAINTS.md | 社會權限層級 | SA-L 權限校驗 |
| FORMAL_VERIFIER.md | 形式化驗證 | 假設驗證 |

### §7.2 約束條件

| 約束類型 | 描述 | 邊界 |
| --- | --- | :--- |
| 異常閾值 | 殘差 > 2σ 標記異常 | z > 2.0 |
| 更新閾值 | 信心度 > 0.8 提議更新 | confidence ≥ 0.8 |
| 原因深度 | 搜尋深度 ≤ 3 | max_depth ≤ 3 |
| 候選數量 | 最多保留 10 個候選 | top 10 |

---

## §8. 版本與演進

| 版本 | 日期 | 變更摘要 |
| --- | :--- | :--- |
| v2.2 | 2026-03 | 初始版本，對應 LOGIC_ENGINE.md §3 |

**演化約束：** 本模組的修改不得違反 NoieLogicAGENTS.md 的不可變核心公理。任何演化提議必須記錄至 EVOLUTION_LOG.md。

---

*Abductive Reasoning Module v2.2 — 異常檢測、原因生成與最短解釋選擇*
*實現因果學習與假設生成的核心能力*
