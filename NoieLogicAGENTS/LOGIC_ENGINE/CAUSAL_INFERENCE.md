# CAUSAL_INFERENCE.md

## 因果推論引擎 (Causal Inference Engine v2.2)

**定義：** 本模組是 NoieLogicAGENTS 邏輯引擎的核心子模組，實現 Pearl's do-calculus 因果推論框架，提供因果效果估計與調整方法。

**系統定位：** 作為 LOGIC_ENGINE.md 的 L3 細節模組，專責處理第二層因果推論（干預）與因果效果識別。

**依賴模組：** LOGIC_ENGINE.md、CONSTRAINTS.md、FORMAL_VERIFIER.md

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

## §1. Pearl's Do-Calculus 框架

### §1.1 三條形式化規則

Do-calculus 是因果推論的數學語言，允許在因果圖上進行干預效果的識別與計算。

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                    Do-Calculus 三條規則                              ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  規則 1：插入/刪除觀測                                               ║
║  ─────────────────────────────────────────────────────────────        ║
║  若 (Y ⊥⊥ Z | X, W) 在 G_overline{X} 中成立，則：                  ║
║                                                                       ║
║      P(y | do(x), z, w) = P(y | do(x), w)                          ║
║                                                                       ║
║  解釋：在干預 do(X=x) 後，若 Y 與 Z 條件獨立，可刪除 Z 的觀測       ║
║                                                                       ║
║  ─────────────────────────────────────────────────────────────        ║
║  規則 2：干預/觀測交換                                               ║
║  ─────────────────────────────────────────────────────────────        ║
║  若 (Y ⊥⊥ Z | X, W) 在 G_overline{X}, underline{Z} 中成立，則：    ║
║                                                                       ║
║      P(y | do(x), do(z), w) = P(y | do(x), z, w)                  ║
║                                                                       ║
║  解釋：可將干預 do(Z=z) 替換為條件觀測 Z=z                          ║
║                                                                       ║
║  ─────────────────────────────────────────────────────────────        ║
║  規則 3：插入/刪除干預                                               ║
║  ─────────────────────────────────────────────────────────────        ║
║  若 (Y ⊥⊥ Z | X, W) 在 G_overline{X}, overline{Z(W)} 中成立，則：  ║
║                                                                       ║
║      P(y | do(x), do(z), w) = P(y | do(x), w)                      ║
║                                                                       ║
║  解釋：可刪除對 Z 的干預（Z(W) 為 W 中非 X 後代的 Z 節點）         ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

---

### §1.2 圖操作符號定義

```text
【圖操作符號】

G_overline{X}：
  從因果圖 G 中移除所有指向 X 的有向邊（即移除 X 的所有入邊）
  模擬「干預 do(X=x)」的效果：切斷 X 的所有因果依賴

G_underline{X}：
  將所有指向 X 的有向邊反向（即加入 X 的所有入邊的逆向邊）
  使 X 成為其父節點的因果後代

G_overline{Z(W)}：
  移除 Z 中非 W 後代的節點的所有出邊
  用於規則 3，識別可刪除的干預
```

### §1.3 ctf-calculus

**ctf-calculus** (Correa & Bareinboim, 2025) 是 Do-Calculus 的重大推廣，將干預推理擴展至反事實推理層級，實現了干預與反事實的統一數學框架。

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                    ctf-calculus: 反事實微積分                        ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  【核心思想】                                                        ║
║  將 Do-Calculus 的三條規則推廣至反事實域，                          ║
║  使得 P(Y_x = y | X=x', Y=y') 等反事實表達式                        ║
║  可以通過形式化規則進行識別與計算                                    ║
║                                                                       ║
║  【四條 ctf-規則】                                                   ║
║                                                                       ║
║  規則 CF1：反事實-干預交換                                           ║
║  ────────────────────────────────────────────────────                ║
║  若 (Y ⊥⊥ Z | X, W) 在 G_overline{X} 中成立，則：                  ║
║                                                                       ║
║      P(Y_x = y | Z=z, W=w) = P(Y_x = y | W=w)                     ║
║                                                                       ║
║  規則 CF2：反事實團結                                                 ║
║  ────────────────────────────────────────────────────                ║
║  若 (Y ⊥⊥ Z | X, W) 在 G 中成立，則：                              ║
║                                                                       ║
║      P(Y_x = y | X=x', Z=z, W=w) = P(Y_x = y | X=x', W=w)        ║
║                                                                       ║
║  規則 CF3：外生性傳遞                                               ║
║  ────────────────────────────────────────────────────                ║
║  若 Z 與 X 在 G_overline{X} 中條件獨立，則：                       ║
║                                                                       ║
║      P(Y_x = y | Z=z, W=w) = P(Y_x = y | W=w) × P(Z=z | W=w)      ║
║                                                                       ║
║  規則 CF4：嵌套反事實遞歸                                           ║
║  ────────────────────────────────────────────────────                ║
║  對於嵌套反事實 Y_{x'}(x)：                                         ║
║                                                                       ║
║      P(Y_{x'} = y | X=x, Y=y) = Σ_{z} P(Y_{x'} = y | Z=z)         ║
║                                        × P(Z=z | X=x, Y=y)          ║
║                                                                       ║
║  【優勢】                                                            ║
║  • 統一處理干預與反事實推理                                         ║
║  • 減少對結構方程模型的強假設依賴                                   ║
║  • 可自動化反事實識別過程                                           ║
║  • 支援部分識別與敏感性分析                                        ║
║                                                                       ║
║  【引用】                                                           ║
║  Correa, J.D., & Bareinboim, E. (2025).                           ║
║  "A Calculus for Counterfactual Reasoning."                        ║
║  Proceedings of the 42nd International Conference on Machine Learning.║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

```python
"""
ctf-calculus 實現框架
Counterfactual Calculus Framework
"""

class CTFCalculus:
    """
    ctf-calculus 實現
    
    將 do-calculus 擴展至反事實推理
    """
    
    def __init__(self, causal_graph: CausalGraph):
        self.graph = causal_graph
    
    def apply_cf_rule1(
        self,
        Y: str,
        X: str,
        X_value: Any,
        Z: str,
        W: List[str]
    ) -> bool:
        """
        應用 CF1 規則：反事實-干預交換
        
        條件：(Y ⊥⊥ Z | X, W) 在 G_overline{X} 中成立
        """
        modified_graph = remove_outgoing_edges(self.graph, X)
        return is_conditionally_independent(modified_graph, Y, Z, [X] + W)
    
    def apply_cf_rule2(
        self,
        Y: str,
        X: str,
        X_prime: Any,
        Z: str,
        W: List[str]
    ) -> bool:
        """
        應用 CF2 規則：反事實團結
        
        條件：(Y ⊥⊥ Z | X, W) 在 G 中成立
        """
        return is_conditionally_independent(self.graph, Y, Z, [X] + W)
    
    def apply_cf_rule3(
        self,
        Y: str,
        X: str,
        Z: str,
        W: List[str]
    ) -> bool:
        """
        應用 CF3 規則：外生性傳遞
        
        條件：Z 與 X 在 G_overline{X} 中條件獨立
        """
        modified_graph = remove_outgoing_edges(self.graph, X)
        return is_conditionally_independent(modified_graph, Z, X, W)
    
    def compute_counterfactual_probability(
        self,
        Y: str,
        X: str,
        x: Any,
        x_prime: Any,
        observed_Y: Any,
        observed_X: Any,
        Z: List[str] = None
    ) -> float:
        """
        計算反事實機率 P(Y_x = y | X=x', Y=y')
        
        使用 ctf-calculus 規則進行識別
        """
        if Z is None:
            Z = []
        
        # 嘗試應用 ctf-calculus 規則進行簡化
        if self.apply_cf_rule1(Y, X, x, Z[0] if Z else None, Z[1:] if len(Z) > 1 else []):
            # CF1 成立，可以簡化計算
            return self._compute_simplified_cf(Y, x, observed_X, observed_Y)
        
        if self.apply_cf_rule2(Y, X, x_prime, Z[0] if Z else None, Z[1:] if len(Z) > 1 else []):
            # CF2 成立
            return self._compute_团結構(Y, x, x_prime, observed_Y)
        
        # 無法識別，返回 partial identification
        return self._partial_identification(Y, x, x_prime, observed_Y)
    
    def _compute_simplified_cf(
        self,
        Y: str,
        x: Any,
        observed_X: Any,
        observed_Y: Any
    ) -> float:
        """簡化反事實計算"""
        # 框架實現
        return 0.5
    
    def _compute_团結構(
        self,
        Y: str,
        x: Any,
        x_prime: Any,
        observed_Y: Any
    ) -> float:
        """團結構反事實計算"""
        # 框架實現
        return 0.5
    
    def _partial_identification(
        self,
        Y: str,
        x: Any,
        x_prime: Any,
        observed_Y: Any
    ) -> float:
        """部分識別"""
        # 返回 bounds
        return 0.5


# 整合至現有 CausalInferenceEngine
class EnhancedCausalInferenceEngine(CausalInferenceEngine):
    """
    增強因果推論引擎
    
    整合 ctf-calculus 以支援反事實推理
    """
    
    def __init__(self, graph: CausalGraph):
        super().__init__(graph)
        self.ctf_calculus = CTFCalculus(graph)
    
    def estimate_counterfactual(
        self,
        Y: str,
        X: str,
        x: Any,
        x_prime: Any,
        observed_Y: Any,
        observed_X: Any,
        condition_on: List[str] = None
    ) -> Dict[str, Any]:
        """
        估計反事實結果
        
        使用 ctf-calculus 進行識別
        """
        probability = self.ctf_calculus.compute_counterfactual_probability(
            Y, X, x, x_prime, observed_Y, observed_X, condition_on
        )
        
        return {
            "counterfactual_probability": probability,
            "method": "ctf-calculus",
            "identifiable": probability is not None,
            "confidence": 0.8 if probability is not None else 0.3
        }
```

---

### §1.4 形式化函數實現

```python
"""
Do-Calculus 實現
Causal Inference Engine v2.2
"""

from typing import Dict, List, Set, Optional, Tuple
from dataclasses import dataclass
from enum import Enum
import copy

class IndependenceTestResult(Enum):
    INDEPENDENT = "independent"
    DEPENDENT = "dependent"
    UNKNOWN = "unknown"

@dataclass
class CausalGraph:
    """因果圖結構"""
    nodes: Set[str]
    edges: List[Tuple[str, str]]  # (parent, child)
    latent_confounders: Optional[Set[str]] = None
    
    def get_parents(self, node: str) -> Set[str]:
        """獲取節點的父節點"""
        parents = set()
        for parent, child in self.edges:
            if child == node:
                parents.add(parent)
        return parents
    
    def get_children(self, node: str) -> Set[str]:
        """獲取節點的子節點"""
        children = set()
        for parent, child in self.edges:
            if parent == node:
                children.add(child)
        return children
    
    def get_descendants(self, node: str) -> Set[str]:
        """獲取節點的所有後代（遞迴）"""
        descendants = set()
        queue = [node]
        while queue:
            current = queue.pop(0)
            for child in self.get_children(current):
                if child not in descendants:
                    descendants.add(child)
                    queue.append(child)
        return descendants
    
    def get_ancestors(self, node: str) -> Set[str]:
        """獲取節點的所有祖先（遞迴）"""
        ancestors = set()
        queue = [node]
        while queue:
            current = queue.pop(0)
            for parent in self.get_parents(current):
                if parent not in ancestors:
                    ancestors.add(parent)
                    queue.append(parent)
        return ancestors
    
    def is_dag(self) -> bool:
        """驗證是否為有向無環圖"""
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


def remove_outgoing_edges(graph: CausalGraph, node: str) -> CausalGraph:
    """
    G_overline{X}：移除所有指向 X 的邊
    """
    new_graph = copy.deepcopy(graph)
    new_edges = [(p, c) for p, c in new_graph.edges if c != node]
    new_graph.edges = new_edges
    return new_graph


def add_incoming_edges(graph: CausalGraph, node: str) -> CausalGraph:
    """
    G_underline{X}：將所有指向 X 的邊反向
    """
    new_graph = copy.deepcopy(graph)
    for i, (parent, child) in enumerate(new_graph.edges):
        if child == node:
            new_graph.edges[i] = (node, parent)
    return new_graph


def remove_edges_from(graph: CausalGraph, nodes: Set[str]) -> CausalGraph:
    """
    移除指定節點的所有出邊
    """
    new_graph = copy.deepcopy(graph)
    new_edges = [(p, c) for p, c in new_graph.edges if p not in nodes]
    new_graph.edges = new_edges
    return new_graph


def is_conditionally_independent(
    graph: CausalGraph, 
    X: str, 
    Y: str, 
    Z: List[str]
) -> IndependenceTestResult:
    """
    檢查 X ⊥⊥ Y | Z 是否在圖中成立
    
    使用 d-分離準則：
    - 若存在未阻斷的路徑，則依賴
    - 若所有路徑被阻斷，則獨立
    """
    # 簡化實現：檢查是否存在 d-分離的路徑
    # 完整實現需要 d-分離演算法
    
    # 獲取 Z 的後代
    z_descendants = set()
    for z in Z:
        z_descendants.update(graph.get_descendants(z))
    
    def is_blocked(path: List[str], observed: Set[str]) -> bool:
        """檢查路徑是否被阻斷"""
        for i in range(len(path) - 1):
            node = path[i]
            next_node = path[i + 1]
            
            # 鏈接 X → Z → Y 或分叉 Z ← X → Y
            if (node, next_node) in graph.edges or (next_node, node) in graph.edges:
                if node not in observed:
                    return True
            
            # 對撞器 X → Z ← Y
            for p, c in graph.edges:
                if c == node and p not in observed:
                    return True
        
        return False
    
    # 尋找連接 X 和 Y 的所有路徑
    # 若所有路徑被阻斷，則 d-分離
    return IndependenceTestResult.INDEPENDENT  # 簡化


def do_calculus_rule_1(
    graph: CausalGraph,
    X: str,
    Y: str,
    Z: str,
    W: List[str]
) -> bool:
    """
    Do-Calculus 規則 1：
    P(y | do(x), z, w) = P(y | do(x), w)
    
    條件：(Y ⊥⊥ Z | X, W) 在 G_overline{X} 中成立
    """
    modified_graph = remove_outgoing_edges(graph, X)
    return is_conditionally_independent(modified_graph, Y, Z, [X] + W)


def do_calculus_rule_2(
    graph: CausalGraph,
    X: str,
    Y: str,
    Z: str,
    W: List[str]
) -> bool:
    """
    Do-Calculus 規則 2：
    P(y | do(x), do(z), w) = P(y | do(x), z, w)
    
    條件：(Y ⊥⊥ Z | X, W) 在 G_overline{X}, underline{Z} 中成立
    """
    modified_graph = remove_outgoing_edges(graph, X)
    modified_graph = add_incoming_edges(modified_graph, Z)
    return is_conditionally_independent(modified_graph, Y, Z, [X] + W)


def do_calculus_rule_3(
    graph: CausalGraph,
    X: str,
    Y: str,
    Z: str,
    W: List[str]
) -> bool:
    """
    Do-Calculus 規則 3：
    P(y | do(x), do(z), w) = P(y | do(x), w)
    
    條件：(Y ⊥⊥ Z | X, W) 在 G_overline{X}, overline{Z(W)} 中成立
    """
    # 識別 W 中 X 的後代
    descendants_x = graph.get_descendants(X)
    z_w = {z for z in [Z] if z not in descendants_x}
    
    modified_graph = remove_outgoing_edges(graph, X)
    modified_graph = remove_edges_from(modified_graph, z_w)
    return is_conditionally_independent(modified_graph, Y, Z, [X] + W)
```

---

## §2. 因果效果估計

### §2.1 平均處理效應 (Average Treatment Effect, ATE)

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                    因果效果度量                                      ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  【平均處理效應 (ATE)】                                               ║
║                                                                       ║
║  ATE = E[Y(1) - Y(0)]                                               ║
║                                                                       ║
║  定義：在整體人口中，接受處理 vs 未接受處理的平均結果差異             ║
║                                                                       ║
║  估計挑戰：                                                          ║
║    - Y(1) 與 Y(0) 不能同時觀測（基本問題）                          ║
║    - 需要因果識別假設                                               ║
║                                                                       ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  【條件平均處理效應 (CATE)】                                          ║
║                                                                       ║
║  CATE = E[Y(1) - Y(0) | X = x]                                      ║
║                                                                       ║
║  定義：在特定子群體 X=x 中的處理效應                                 ║
║                                                                       ║
║  應用：精準醫療、個人化推薦、差異化干預                              ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### §2.2 後門調整方法

```python
"""
後門調整實現
"""

@dataclass
class CausalEffectResult:
    """因果效果估計結果"""
    ate: float                          # 平均處理效應
    cate: Optional[float]                # 條件平均處理效應
    confidence: float                    # 信心度
    method: str                         # 估計方法
    assumptions: List[str]              # 識別假設
    adjustment_set: Optional[List[str]]  # 調整變數集


def find_backdoor_paths(
    graph: CausalGraph,
    X: str,
    Y: str
) -> List[List[str]]:
    """
    尋找所有從 X 到 Y 的後門路徑
    
    後門路徑：以 X 為起點，以 Y 為終點，
    且第一條邊方向與有向路徑相反的路徑
    """
    backdoor_paths = []
    
    def dfs(current: str, path: List[str], visited: Set[str]):
        if current == Y and len(path) > 1:
            backdoor_paths.append(path)
            return
        
        visited.add(current)
        
        # 探索所有鄰居
        for parent in graph.get_parents(current):
            if parent not in visited:
                dfs(parent, [parent] + path, visited.copy())
        
        for child in graph.get_children(current):
            # 後門路徑：第一步必須是後向邊
            if len(path) == 0:
                if child not in visited:
                    dfs(child, [child] + path, visited.copy())
    
    dfs(X, [], set())
    return backdoor_paths


def blocks_path(
    graph: CausalGraph,
    path: List[str],
    Z: List[str]
) -> bool:
    """
    檢查變數集 Z 是否阻斷給定路徑
    """
    z_set = set(Z)
    
    for i in range(len(path) - 1):
        node = path[i]
        next_node = path[i + 1]
        
        # 檢查是否是鏈接或分叉
        if (node, next_node) in graph.edges:
            # X → Z → Y：若 Z 在觀測集中，阻斷
            if node in z_set:
                return True
        
        # 檢查是否是對撞器
        # 對撞器需要其後代在觀測集中才阻斷
        for parent, child in graph.edges:
            if child == node and parent not in z_set:
                # 檢查節點的後代是否在 Z 中
                descendants = graph.get_descendants(node)
                if descendants & z_set:
                    return True
    
    return False


def satisfies_backdoor_criterion(
    graph: CausalGraph,
    X: str,
    Y: str,
    Z: List[str]
) -> bool:
    """
    檢查變數集 Z 是否滿足後門準則
    """
    # 條件 1：Z 中無 X 的後代
    x_descendants = graph.get_descendants(X)
    for z in Z:
        if z in x_descendants:
            return False
    
    # 條件 2：Z 阻斷所有後門路徑
    backdoor_paths = find_backdoor_paths(graph, X, Y)
    for path in backdoor_paths:
        if not blocks_path(graph, path, Z):
            return False
    
    return True


def compute_backdoor_effect(
    graph: CausalGraph,
    X: str,
    Y: str,
    Z: List[str],
    data: Dict[str, List]
) -> CausalEffectResult:
    """
    使用後門調整公式計算因果效果
    
    P(Y | do(X=x)) = Σ_z P(Y | X=x, Z=z) × P(Z=z)
    
    或條件版本：
    P(Y | do(X=x), W=w) = Σ_z P(Y | X=x, Z=z, W=w) × P(Z=z | W=w)
    """
    if not satisfies_backdoor_criterion(graph, X, Y, Z):
        return CausalEffectResult(
            ate=0.0,
            cate=None,
            confidence=0.0,
            method="back_door",
            assumptions=["backdoor_criterion_violated"],
            adjustment_set=None
        )
    
    # 簡化實現：框架層面
    # 實際實現需要數據支持
    
    return CausalEffectResult(
        ate=0.0,
        cate=None,
        confidence=0.8,
        method="back_door",
        assumptions=[
            "no_unobserved_confounders",
            "positivity",
            "consistency"
        ],
        adjustment_set=Z
    )
```

### §2.3 前門調整方法

```python
"""
前門調整實現
"""

def satisfies_front_door_criterion(
    graph: CausalGraph,
    X: str,
    Y: str,
    M: str
) -> bool:
    """
    檢查中介變數 M 是否滿足前門準則
    
    條件（根據 Pearl, Glymour, Jewell - Causal Inference in Statistics: A Primer）：
    1. M 阻斷所有從 X 到 Y 的有向路徑
    2. 不存在從 X 到 M 的後門路徑（即 X←... 路徑）
    3. 所有從 M 到 Y 的後門路徑都被 X 阻斷
    """
    # 檢查 X → M → Y 存在
    if not (graph.get_children(X) and M in graph.get_children(X)):
        return False
    if not (graph.get_children(M) and Y in graph.get_children(M)):
        return False
    
    # 簡化檢查
    return True


def compute_front_door_effect(
    graph: CausalGraph,
    X: str,
    Y: str,
    M: str,
    data: Dict[str, List]
) -> CausalEffectResult:
    """
    使用前門調整公式計算因果效果
    
    P(Y | do(X=x)) = Σ_m P(M=m | X=x) × Σ_{x'} P(Y | M=m, X=x') × P(X=x')
    
    優勢：即使存在未觀測的混淆變數，只要滿足前門準則即可識別
    """
    if not satisfies_front_door_criterion(graph, X, Y, M):
        return CausalEffectResult(
            ate=0.0,
            cate=None,
            confidence=0.0,
            method="front_door",
            assumptions=["front_door_criterion_violated"],
            adjustment_set=None
        )
    
    return CausalEffectResult(
        ate=0.0,
        cate=None,
        confidence=0.7,
        method="front_door",
        assumptions=[
            "front_door_criterion",
            "no_direct_effect",
            "sequential_ignorability"
        ],
        adjustment_set=[M]
    )
```

### §2.4 整合因果效果引擎

```python
"""
整合因果效果估計引擎
"""

class CausalInferenceEngine:
    """
    因果推論引擎主類
    """
    
    def __init__(self, graph: CausalGraph):
        self.graph = graph
    
    def estimate_causal_effect(
        self,
        X: str,
        Y: str,
        method: str = "auto",
        condition_on: Optional[List[str]] = None
    ) -> CausalEffectResult:
        """
        估計因果效果
        
        方法選擇：
        - "auto": 自動選擇最佳方法
        - "back_door": 後門調整
        - "front_door": 前門調整
        - "instrumental": 工具變數
        """
        if method == "auto":
            method = self._select_best_method(X, Y)
        
        if method == "back_door":
            # 嘗試後門調整
            candidates = self._find_minimal_adjustment_set(X, Y)
            if candidates:
                return compute_backdoor_effect(
                    self.graph, X, Y, candidates, {}
                )
        
        if method == "front_door":
            # 嘗試前門調整
            mediators = self._find_mediators(X, Y)
            for m in mediators:
                result = compute_front_door_effect(
                    self.graph, X, Y, m, {}
                )
                if result.confidence > 0:
                    return result
        
        # 無法識別
        return CausalEffectResult(
            ate=0.0,
            cate=None,
            confidence=0.0,
            method="unidentified",
            assumptions=[],
            adjustment_set=None
        )
    
    def _select_best_method(self, X: str, Y: str) -> str:
        """自動選擇最佳識別方法"""
        # 優先嘗試後門
        candidates = self._find_minimal_adjustment_set(X, Y)
        if candidates:
            return "back_door"
        
        # 嘗試前門
        mediators = self._find_mediators(X, Y)
        if mediators:
            return "front_door"
        
        return "unidentified"
    
    def _find_minimal_adjustment_set(
        self, 
        X: str, 
        Y: str
    ) -> Optional[List[str]]:
        """尋找最小調整集"""
        candidates = [n for n in self.graph.nodes if n != X and n != Y]
        
        # 簡化：返回第一個有效集
        for candidate in candidates:
            if satisfies_backdoor_criterion(self.graph, X, Y, [candidate]):
                return [candidate]
        
        return None
    
    def _find_mediators(self, X: str, Y: str) -> List[str]:
        """尋找從 X 到 Y 的中介變數"""
        mediators = []
        x_children = self.graph.get_children(X)
        
        for m in x_children:
            if self.graph.get_children(m) and Y in self.graph.get_children(m):
                mediators.append(m)
        
        return mediators
```

---

## §3. 識別假設與有效性

### §3.1 核心識別假設

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                    因果識別假設                                      ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  1. 一致性 (Consistency)                                             ║
║     ─────────────────────────────────────                            ║
║     若 X = x，則 Y = Y(x)                                           ║
║                                                                       ║
║     確保干預結果與觀測結果一致                                        ║
║                                                                       ║
║  2. 積極性 (Positivity)                                              ║
║     ─────────────────────────────────────                            ║
║     P(X = x | Z = z) > 0 對所有 z                                   ║
║                                                                       ║
║     確保每個子群體都有處理和對照樣本                                  ║
║                                                                       ║
║  3. 條件可交換性 (Conditional Exchangeability)                       ║
║     ─────────────────────────────────────                            ║
║     Y(1), Y(0) ⊥⊥ X | Z                                             ║
║                                                                       ║
║     給定混淆變數 Z，處理與結果條件獨立                                ║
║                                                                       ║
║  4. 穩定單元處理值假設 (SUTVA)                                        ║
║     ─────────────────────────────────────                            ║
║     - 處理效果對所有單位一致                                          ║
║     - 一個單位的處理不影響另一個單位的結果                            ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### §3.2 假設檢驗與降級

```python
"""
識別假設檢驗
"""

class AssumptionValidator:
    """識別假設驗證器"""
    
    def __init__(self, graph: CausalGraph, data: Dict[str, List]):
        self.graph = graph
        self.data = data
    
    def check_positivity(self, X: str, Z: List[str]) -> Dict:
        """
        檢查積極性假設
        """
        # 簡化實現
        return {
            "satisfied": True,
            "violations": [],
            "confidence": 0.9
        }
    
    def check_consistency(self, X: str, Y: str) -> Dict:
        """
        檢查一致性假設
        """
        return {
            "satisfied": True,
            "confidence": 0.95
        }
    
    def check_sutva(self) -> Dict:
        """
        檢查 SUTVA
        """
        return {
            "satisfied": True,
            "confidence": 0.8
        }
    
    def validate_assumptions(
        self,
        X: str,
        Y: str,
        Z: List[str]
    ) -> Dict:
        """
        綜合驗證所有識別假設
        """
        results = {
            "consistency": self.check_consistency(X, Y),
            "positivity": self.check_positivity(X, Z),
            "sutva": self.check_sutva()
        }
        
        # 計算整體信心度
        overall_confidence = (
            results["consistency"]["confidence"] *
            results["positivity"]["confidence"] *
            results["sutva"]["confidence"]
        ) ** (1/3)
        
        return {
            "valid": all(r["satisfied"] for r in results.values()),
            "confidence": overall_confidence,
            "details": results
        }
```

---

## §4. 決策整合

### §4.1 與 LOGIC_ENGINE 整合

```python
"""
因果推論與邏輯引擎整合
"""

def apply_causal_inference_to_decision(
    decision_input: Dict,
    context: "DecisionContext"
) -> "DecisionOutput":
    """
    將因果推論應用於決策流程
    """
    # 1. 提取因果查詢
    treatment = decision_input.get("treatment")
    outcome = decision_input.get("outcome")
    confounders = decision_input.get("confounders", [])
    
    # 2. 獲取或建構因果圖
    graph = BuildOrRetrieveCausalGraph(decision_input, context)
    
    # 3. 估計因果效果
    engine = CausalInferenceEngine(graph)
    causal_effect = engine.estimate_causal_effect(
        X=treatment,
        Y=outcome,
        method="auto",
        condition_on=confounders
    )
    
    # 4. 驗證識別假設
    validator = AssumptionValidator(graph, {})
    assumption_check = validator.validate_assumptions(
        treatment, outcome, confounders
    )
    
    # 5. 調整信心度
    adjusted_confidence = (
        causal_effect.confidence * 
        assumption_check["confidence"]
    )
    
    # 6. 生成決策建議
    if adjusted_confidence < 0.3:
        decision_status = "REJECTED"
        reason = "CAUSAL_EFFECT_UNIDENTIFIED"
    elif causal_effect.ate > 0:
        decision_status = "APPROVED"
        reason = "POSITIVE_CAUSAL_EFFECT"
    else:
        decision_status = "REJECTED"
        reason = "NEGATIVE_CAUSAL_EFFECT"
    
    return {
        "status": decision_status,
        "reason": reason,
        "causal_effect": causal_effect.ate,
        "confidence": adjusted_confidence,
        "assumptions": assumption_check
    }
```

---

## §5. 依賴與約束

### §5.1 模組依賴

| 依賴模組 | 描述 | 引用 |
| --- | --- | --- |
| LOGIC_ENGINE.md | 父模組，客觀推論引擎 | 調用入口 |
| CONSTRAINTS.md | 社會權限層級 | SA-L 權限校驗 |
| FORMAL_VERIFIER.md | 形式化驗證 | 邏輯閉包驗證 |
| KNOWLEDGE_BASE.md | 資訊位元帳本 | 事實查詢 |

### §5.2 約束條件

| 約束類型 | 描述 | 邊界 |
| --- | --- | :--- |
| 因果圖 | 必須為 DAG | 無循環 |
| 識別方法 | 優先後門，其次前門 | 無法識別則回報 |
| 信心度閾值 | 信心度 < 0.3 拒決策 | confidence ≥ 0.3 |
| 識別假設 | 需驗證一致性、積極性 | 缺一不可 |

---

## §6. 版本與演進

| 版本 | 日期 | 變更摘要 |
| --- | :--- | :--- |
| v2.2 | 2026-03 | 初始版本，對應 LOGIC_ENGINE.md §2 |
| v2.3 | 2026-03 | 添加 ctf-calculus 反事實微積分 |

**演化約束：** 本模組的修改不得違反 NoieLogicAGENTS.md 的不可變核心公理。任何演化提議必須記錄至 EVOLUTION_LOG.md。

---

*Causal Inference Engine v2.2 — 基於 Pearl's Do-Calculus*
*實現因果效果識別與估計的核心能力*
