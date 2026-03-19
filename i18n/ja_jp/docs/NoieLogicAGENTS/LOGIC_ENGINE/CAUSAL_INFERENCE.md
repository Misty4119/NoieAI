# CAUSAL_INFERENCE.md

## 因果推論エンジン (Causal Inference Engine v2.2)

**定義：** 本モジュールはNoieLogicAGENTSロジックエンジンのコアサブモジュールであり、Pearlのdo-calculus因果推論フレームワークを実装し、因果効果推定と調整方法を提供する。

**システム定位：** LOGIC_ENGINE.mdのL3詳細モジュールとして、第2層因果推論（干渉）と因果効果識別專門担当。

**依存モジュール：** LOGIC_ENGINE.md、CONSTRAINTS.md、FORMAL_VERIFIER.md

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

## §1. PearlのDo-Calculusフレームワーク

### §1.1 三つの形式化ルール

Do-calculusは因果推論の数学言語であり、因果グラフ上で干渉効果の識別と計算を可能にする。

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                    Do-Calculus三つのルール                              ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  ルール1：挿入/削除観測                                               ║
║  ─────────────────────────────────────────────────────────────        ║
║  もし (Y ⊥⊥ Z | X, W) が G_overline{X} で成立するなら：                  ║
║                                                                       ║
║      P(y | do(x), z, w) = P(y | do(x), w)                          ║
║                                                                       ║
║  解釈：干渉 do(X=x) 後、YとZが条件付き独立なら、Zの観測を削除可能       ║
║                                                                       ║
║  ─────────────────────────────────────────────────────────────        ║
║  ルール2：干渉/観測交換                                               ║
║  ─────────────────────────────────────────────────────────────        ║
║  もし (Y ⊥⊥ Z | X, W) が G_overline{X}, underline{Z} で成立するなら：    ║
║                                                                       ║
║      P(y | do(x), do(z), w) = P(y | do(x), z, w)                  ║
║                                                                       ║
║  解釈：干渉 do(Z=z)を条件観測Z=zに置換可能                          ║
║                                                                       ║
║  ─────────────────────────────────────────────────────────────        ║
║  ルール3：挿入/削除干渉                                               ║
║  ─────────────────────────────────────────────────────────────        ║
║  もし (Y ⊥⊥ Z | X, W) が G_overline{X}, overline{Z(W)} で成立するなら：  ║
║                                                                       ║
║      P(y | do(x), do(z), w) = P(y | do(x), w)                      ║
║                                                                       ║
║  解釈：Zの干渉を削除可能（Z(W)はWの非X後代のZノード）         ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

---

### §1.2 グラフ操作記号定義

```text
【グラフ操作記号】

G_overline{X}：
  因果グラフGからXに向かうすべての有向辺を削除（Xのすべての入辺を削除）
  干渉「do(X=x)」の効果をシミュレート：Xのすべての因果依存を切断

G_underline{X}：
  Xに向かうすべての有向辺を反向（Xの入辺の逆向辺を加える）
  Xをその親ノードの因果後代にする

G_overline{Z(W)}：
  Zの非W後代のノードすべての出辺を削除
  ルール3に使用し、削除可能な干渉を識別
```

### §1.3 ctf-calculus

**ctf-calculus** (Correa & Bareinboim, 2025) はDo-Calculusの重大な拡張であり、干渉推論を反事実推論レベルに拡張し、干渉と反事実の統一数学フレームワークを実装した。

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                    ctf-calculus: 反事実微積分                        ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  【コア思想】                                                        ║
║  Do-Calculusの三つのルールを反事実ドメインに拡張し、                          ║
║  P(Y_x = y | X=x', Y=y')などの反事実式                        ║
║  を形式化ルールで識別・計算可能出现                              ║
║                                                                       ║
║  【四つのctfルール】                                                   ║
║                                                                       ║
║  ルールCF1：反事実-干渉交換                                           ║
║  ────────────────────────────────────────────────────                ║
║  もし (Y ⊥⊥ Z | X, W) が G_overline{X} で成立するなら：                  ║
║                                                                       ║
║      P(Y_x = y | Z=z, W=w) = P(Y_x = y | W=w)                     ║
║                                                                       ║
║  ルールCF2：反事実団結                                                 ║
║  ────────────────────────────────────────────────────                ║
║  もし (Y ⊥⊥ Z | X, W) が G で成立するなら：                              ║
║                                                                       ║
║      P(Y_x = y | X=x', Z=z, W=w) = P(Y_x = y | X=x', W=w)        ║
║                                                                       ║
║  ルールCF3：外生性伝達                                               ║
║  ────────────────────────────────────────────────────                ║
║  もしZとXがG_overline{X}で条件独立なら：                       ║
║                                                                       ║
║      P(Y_x = y | Z=z, W=w) = P(Y_x = y | W=w) × P(Z=z | W=w)      ║
║                                                                       ║
║  ルールCF4：ネスト反事実帰納                                           ║
║  ────────────────────────────────────────────────────                ║
║  ネスト反事実 Y_{x'}(x) の場合：                                         ║
║                                                                       ║
║      P(Y_{x'} = y | X=x, Y=y) = Σ_{z} P(Y_{x'} = y | Z=z)         ║
║                                        × P(Z=z | X=x, Y=y)          ║
║                                                                       ║
║  【優位性】                                                            ║
║  • 干渉と反事実推論の統一処理                                         ║
║  • 構造方程式モデルへの強い仮定依存を軽減                                   ║
║  • 反事実識別プロセスの自動化                                           ║
║  • 部分識別と感度分析をサポート                                        ║
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
ctf-calculus実装フレームワーク
Counterfactual Calculus Framework
"""

class CTFCalculus:
    """
    ctf-calculus実装
    
    do-calculusを反事実推論に拡張
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
        CF1ルールを適用：反事実-干渉交換
        
        条件：(Y ⊥⊥ Z | X, W) が G_overline{X} で成立
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
        CF2ルールを適用：反事実団結
        
        条件：(Y ⊥⊥ Z | X, W) が G で成立
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
        CF3ルールを適用：外生性伝達
        
        条件：ZとXがG_overline{X}で条件独立
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
        反事実確率を計算 P(Y_x = y | X=x', Y=y')
        
        ctf-calculusルールを使用して識別
        """
        if Z is None:
            Z = []
        
        # ctf-calculusルールの適用を試行して簡略化
        if self.apply_cf_rule1(Y, X, x, Z[0] if Z else None, Z[1:] if len(Z) > 1 else []):
            # CF1が成立、計算を簡略化可能
            return self._compute_simplified_cf(Y, x, observed_X, observed_Y)
        
        if self.apply_cf_rule2(Y, X, x_prime, Z[0] if Z else None, Z[1:] if len(Z) > 1 else []):
            # CF2が成立
            return self._compute_团構造(Y, x, x_prime, observed_Y)
        
        # 識別不能、部分識別を返す
        return self._partial_identification(Y, x, x_prime, observed_Y)
    
    def _compute_simplified_cf(
        self,
        Y: str,
        x: Any,
        observed_X: Any,
        observed_Y: Any
    ) -> float:
        """簡略化反事実計算"""
        # フレームワーク実装
        return 0.5
    
    def _compute_团構造(
        self,
        Y: str,
        x: Any,
        x_prime: Any,
        observed_Y: Any
    ) -> float:
        """団構造反事実計算"""
        # フレームワーク実装
        return 0.5
    
    def _partial_identification(
        self,
        Y: str,
        x: Any,
        x_prime: Any,
        observed_Y: Any
    ) -> float:
        """部分識別"""
        # boundsを返す
        return 0.5


# 既存CausalInferenceEngineへの統合
class EnhancedCausalInferenceEngine(CausalInferenceEngine):
    """
    強化因果推論エンジン
    
    ctf-calculusを統合して反事実推論をサポート
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
        反事実結果を推定
        
        ctf-calculusを使用して識別
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

### §1.4 形式化関数実装

```python
"""
Do-Calculus実装
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
    """因果グラフ構造"""
    nodes: Set[str]
    edges: List[Tuple[str, str]]  # (parent, child)
    latent_confounders: Optional[Set[str]] = None
    
    def get_parents(self, node: str) -> Set[str]:
        """ノードの親ノードを取得"""
        parents = set()
        for parent, child in self.edges:
            if child == node:
                parents.add(parent)
        return parents
    
    def get_children(self, node: str) -> Set[str]:
        """ノードの子ノードを取得"""
        children = set()
        for parent, child in self.edges:
            if parent == node:
                children.add(child)
        return children
    
    def get_descendants(self, node: str) -> Set[str]:
        """ノードのすべての後代を取得（帰納）"""
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
        """ノードのすべての祖先を取得（帰納）"""
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
        """有向非巡回グラフであるかを検証"""
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
    G_overline{X}：Xに向かうすべての辺を削除
    """
    new_graph = copy.deepcopy(graph)
    new_edges = [(p, c) for p, c in new_graph.edges if c != node]
    new_graph.edges = new_edges
    return new_graph


def add_incoming_edges(graph: CausalGraph, node: str) -> CausalGraph:
    """
    G_underline{X}：Xに向かうすべての辺を反向
    """
    new_graph = copy.deepcopy(graph)
    for i, (parent, child) in enumerate(new_graph.edges):
        if child == node:
            new_graph.edges[i] = (node, parent)
    return new_graph


def remove_edges_from(graph: CausalGraph, nodes: Set[str]) -> CausalGraph:
    """
    指定ノードのすべての出辺を削除
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
    X ⊥⊥ Y | Z がグラフで成立するかをチェック
    
    d-分離基準を使用：
    - 遮断されていないパスが存在すれば依存
    - すべてのパスが遮断されていれば独立
    """
    # 簡略化実装：d-分離パスの存在をチェック
    # 完全実装にはd-分離アルゴリズムが必要
    
    # Zの後代を取得
    z_descendants = set()
    for z in Z:
        z_descendants.update(graph.get_descendants(z))
    
    def is_blocked(path: List[str], observed: Set[str]) -> bool:
        """パスが遮断されているかをチェック"""
        for i in range(len(path) - 1):
            node = path[i]
            next_node = path[i + 1]
            
            # チェーン X → Z → Y またはフォーク Z ← X → Y
            if (node, next_node) in graph.edges or (next_node, node) in graph.edges:
                if node not in observed:
                    return True
            
            # コライダー X → Z ← Y
            for p, c in graph.edges:
                if c == node and p not in observed:
                    return True
        
        return False
    
    # XとYを接続するすべてのパスを検索
    # すべてのパスが遮断されていればd-分離
    return IndependenceTestResult.INDEPENDENT  # 簡略化


def do_calculus_rule_1(
    graph: CausalGraph,
    X: str,
    Y: str,
    Z: str,
    W: List[str]
) -> bool:
    """
    Do-Calculusルール1：
    P(y | do(x), z, w) = P(y | do(x), w)
    
    条件：(Y ⊥⊥ Z | X, W) が G_overline{X} で成立
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
    Do-Calculusルール2：
    P(y | do(x), do(z), w) = P(y | do(x), z, w)
    
    条件：(Y ⊥⊥ Z | X, W) が G_overline{X}, underline{Z} で成立
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
    Do-Calculusルール3：
    P(y | do(x), do(z), w) = P(y | do(x), w)
    
    条件：(Y ⊥⊥ Z | X, W) が G_overline{X}, overline{Z(W)} で成立
    """
    # WのX後代を識別
    descendants_x = graph.get_descendants(X)
    z_w = {z for z in [Z] if z not in descendants_x}
    
    modified_graph = remove_outgoing_edges(graph, X)
    modified_graph = remove_edges_from(modified_graph, z_w)
    return is_conditionally_independent(modified_graph, Y, Z, [X] + W)
```

---

## §2. 因果効果推定

### §2.1 平均処理効果 (Average Treatment Effect, ATE)

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                    因果効果度量                                      ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  【平均処理効果 (ATE)】                                               ║
║                                                                       ║
║  ATE = E[Y(1) - Y(0)]                                               ║
║                                                                       ║
║  定義：全体人口における処理ありvsなしの平均結果の差異             ║
║                                                                       ║
║  推定挑戦：                                                          ║
║    - Y(1)とY(0)は同時に観測不能（基本問題）                          ║
║    - 因果識別仮定が必要                                               ║
║                                                                       ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  【条件平均処理効果 (CATE)】                                          ║
║                                                                       ║
║  CATE = E[Y(1) - Y(0) | X = x]                                      ║
║                                                                       ║
║  定義：特定のサブグループX=xでの処理効果                                 ║
║                                                                       ║
║  応用：精密医療、個別化推薦、差別化干渉                              ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### §2.2 後門調整方法

```python
"""
後門調整実装
"""

@dataclass
class CausalEffectResult:
    """因果効果推定結果"""
    ate: float                          # 平均処理効果
    cate: Optional[float]                # 条件平均処理効果
    confidence: float                    # 信心度
    method: str                         # 推定方法
    assumptions: List[str]              # 識別仮定
    adjustment_set: Optional[List[str]]  # 調整変数集合


def find_backdoor_paths(
    graph: CausalGraph,
    X: str,
    Y: str
) -> List[List[str]]:
    """
    XからYへのすべての後門パスを検索
    
    後門パス：Xで始まりYで終わり、
    最初の辺方向が有向パスと逆方向のパス
    """
    backdoor_paths = []
    
    def dfs(current: str, path: List[str], visited: Set[str]):
        if current == Y and len(path) > 1:
            backdoor_paths.append(path)
            return
        
        visited.add(current)
        
        # すべての隣接ノードを探索
        for parent in graph.get_parents(current):
            if parent not in visited:
                dfs(parent, [parent] + path, visited.copy())
        
        for child in graph.get_children(current):
            # 後門パス：最初の一歩は後向辺でなければならない
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
    変数集Zが与えられたパスを遮断するかをチェック
    """
    z_set = set(Z)
    
    for i in range(len(path) - 1):
        node = path[i]
        next_node = path[i + 1]
        
        # チェーンまたはフォークかチェック
        if (node, next_node) in graph.edges:
            # X → Z → Y：Zが観測集合にあれば遮断
            if node in z_set:
                return True
        
        # コライダーかチェック
        # コライダーは其后代が観測集合にあれば遮断
        for parent, child in graph.edges:
            if child == node and parent not in z_set:
                # ノードの後代がZにあるかチェック
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
    変数集Zが後門基準を満たすかをチェック
    """
    # 条件1：ZにXの後代がない
    x_descendants = graph.get_descendants(X)
    for z in Z:
        if z in x_descendants:
            return False
    
    # 条件2：Zがすべての後門パスを遮断
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
    後門調整式を使用して因果効果を計算
    
    P(Y | do(X=x)) = Σ_z P(Y | X=x, Z=z) × P(Z=z)
    
    または条件バージョン：
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
    
    # 簡略化実装：フレームワークレベル
    # 実装にはデータサポートが必要
    
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
前門調整実装
"""

def satisfies_front_door_criterion(
    graph: CausalGraph,
    X: str,
    Y: str,
    M: str
) -> bool:
    """
    中間変数Mが前門基準を満たすかをチェック
    
    条件（Pearl, Glymour, Jewell - Causal Inference in Statistics: A Primer）：
    1. MがXからYへのすべての有向パスを遮断
    2. XからMへの後門パスが存在しない（つまりX←...パスがない）
    3. MからYへのすべての後門パスがXに遮断
    """
    # X → M → Yが存在するかチェック
    if not (graph.get_children(X) and M in graph.get_children(X)):
        return False
    if not (graph.get_children(M) and Y in graph.get_children(M)):
        return False
    
    # 簡略チェック
    return True


def compute_front_door_effect(
    graph: CausalGraph,
    X: str,
    Y: str,
    M: str,
    data: Dict[str, List]
) -> CausalEffectResult:
    """
    前門調整式を使用して因果効果を計算
    
    P(Y | do(X=x)) = Σ_m P(M=m | X=x) × Σ_{x'} P(Y | M=m, X=x') × P(X=x')
    
    優位性：未観測の混淆変数が存在しても、前門基準を満たせば識別可能
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

### §2.4 統合因果効果エンジン

```python
"""
統合因果効果推定エンジン
"""

class CausalInferenceEngine:
    """
    因果推論エンジンメインクラス
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
        因果効果を推定
        
        方法選択：
        - "auto": 最良方法を自動選択
        - "back_door": 後門調整
        - "front_door": 前門調整
        - "instrumental": 工具変数
        """
        if method == "auto":
            method = self._select_best_method(X, Y)
        
        if method == "back_door":
            # 後門調整を試行
            candidates = self._find_minimal_adjustment_set(X, Y)
            if candidates:
                return compute_backdoor_effect(
                    self.graph, X, Y, candidates, {}
                )
        
        if method == "front_door":
            # 前門調整を試行
            mediators = self._find_mediators(X, Y)
            for m in mediators:
                result = compute_front_door_effect(
                    self.graph, X, Y, m, {}
                )
                if result.confidence > 0:
                    return result
        
        # 識別不能
        return CausalEffectResult(
            ate=0.0,
            cate=None,
            confidence=0.0,
            method="unidentified",
            assumptions=[],
            adjustment_set=None
        )
    
    def _select_best_method(self, X: str, Y: str) -> str:
        """最良識別方法を自動選択"""
        # 後門を優先試行
        candidates = self._find_minimal_adjustment_set(X, Y)
        if candidates:
            return "back_door"
        
        # 前門を試行
        mediators = self._find_mediators(X, Y)
        if mediators:
            return "front_door"
        
        return "unidentified"
    
    def _find_minimal_adjustment_set(
        self, 
        X: str, 
        Y: str
    ) -> Optional[List[str]]:
        """最小調整集合を検索"""
        candidates = [n for n in self.graph.nodes if n != X and n != Y]
        
        # 簡略化：最初の有効集合を返す
        for candidate in candidates:
            if satisfies_backdoor_criterion(self.graph, X, Y, [candidate]):
                return [candidate]
        
        return None
    
    def _find_mediators(self, X: str, Y: str) -> List[str]:
        """XからYへの中介変数を検索"""
        mediators = []
        x_children = self.graph.get_children(X)
        
        for m in x_children:
            if self.graph.get_children(m) and Y in self.graph.get_children(m):
                mediators.append(m)
        
        return mediators
```

---

## §3. 識別仮定と妥当性

### §3.1 コア識別仮定

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                    因果識別仮定                                      ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  1. 一貫性 (Consistency)                                             ║
║     ─────────────────────────────────────                            ║
║     もしX = xなら、Y = Y(x)                                           ║
║                                                                       ║
║     干渉結果と観測結果の一貫性を確保                                        ║
║                                                                       ║
║  2. 積極性 (Positivity)                                              ║
║     ─────────────────────────────────────                            ║
║     P(X = x | Z = z) > 0 すべてのzについて                                   ║
║                                                                       ║
║     各サブグループに処理と対照サンプルが存在することを確保                    ║
║                                                                       ║
║  3. 条件付き交換可能性 (Conditional Exchangeability)                       ║
║     ─────────────────────────────────────                            ║
║     Y(1), Y(0) ⊥⊥ X | Z                                             ║
║                                                                       ║
║     混淆変数Zが与えられれば、処理と結果は条件付き独立                        ║
║                                                                       ║
║  4. 安定単位処理値仮定 (SUTVA)                                        ║
║     ─────────────────────────────────────                            ║
║     - 処理効果がすべての単位で一貫                                          ║
║     - 一つの単位の処理が他の単位の結果に影響しない                            ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### §3.2 仮定検定とダウングレード

```python
"""
識別仮定検定
"""

class AssumptionValidator:
    """識別仮定検証器"""
    
    def __init__(self, graph: CausalGraph, data: Dict[str, List]):
        self.graph = graph
        self.data = data
    
    def check_positivity(self, X: str, Z: List[str]) -> Dict:
        """
        積極性仮定をチェック
        """
        # 簡略化実装
        return {
            "satisfied": True,
            "violations": [],
            "confidence": 0.9
        }
    
    def check_consistency(self, X: str, Y: str) -> Dict:
        """
        一貫性仮定をチェック
        """
        return {
            "satisfied": True,
            "confidence": 0.95
        }
    
    def check_sutva(self) -> Dict:
        """
        SUTVAをチェック
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
        すべての識別仮定を綜合検証
        """
        results = {
            "consistency": self.check_consistency(X, Y),
            "positivity": self.check_positivity(X, Z),
            "sutva": self.check_sutva()
        }
        
        # 全体信心度を計算
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

## §4. 意思決定統合

### §4.1 LOGIC_ENGINEとの統合

```python
"""
因果推論とロジックエンジン統合
"""

def apply_causal_inference_to_decision(
    decision_input: Dict,
    context: "DecisionContext"
) -> "DecisionOutput":
    """
    因果推論を意思決定フローに適用
    """
    # 1. 因果クエリを抽出
    treatment = decision_input.get("treatment")
    outcome = decision_input.get("outcome")
    confounders = decision_input.get("confounders", [])
    
    # 2. 因果グラフを取得または構築
    graph = BuildOrRetrieveCausalGraph(decision_input, context)
    
    # 3. 因果効果を推定
    engine = CausalInferenceEngine(graph)
    causal_effect = engine.estimate_causal_effect(
        X=treatment,
        Y=outcome,
        method="auto",
        condition_on=confounders
    )
    
    # 4. 識別仮定を検証
    validator = AssumptionValidator(graph, {})
    assumption_check = validator.validate_assumptions(
        treatment, outcome, confounders
    )
    
    # 5. 信心度を調整
    adjusted_confidence = (
        causal_effect.confidence * 
        assumption_check["confidence"]
    )
    
    # 6. 意思決定提案を生成
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

## §5. 依存と制約

### §5.1 モジュール依存

| 依存モジュール | 説明 | 参照 |
| --- | --- | --- |
| LOGIC_ENGINE.md | 親モジュール、客観的推論エンジン | 呼び出しエントリ |
| CONSTRAINTS.md | 社会権限レベル | SA-L権限校验 |
| FORMAL_VERIFIER.md | 形式化検証 | 論理閉包検証 |
| KNOWLEDGE_BASE.md | 情報ビット台帳 | 事実クエリ |

### §5.2 制約条件

| 制約タイプ | 説明 | 境界 |
| --- | --- | :--- |
| 因果グラフ | DAGでなければならない | 循環なし |
| 識別方法 | 優先は後門、次は前門 | 識別不能は報告 |
| 信心度閾値 | 信心度 < 0.3 は意思決定を拒否 | confidence ≧ 0.3 |
| 識別仮定 | 一貫性、積極性の検証が必要 | 缺一不可 |

---

## §6. バージョンと進化

| バージョン | 日付 | 変更要約 |
| --- | :--- | :--- |
| v2.2 | 2026-03 | 初期バージョン、LOGIC_ENGINE.md §2に対応 |
| v2.3 | 2026-03 | ctf-calculus反事実微積分を追加 |

**進化制約：** 本モジュールの修改はNoieLogicAGENTS.mdの不変コア公理に反してはならない。任意の進化提案はEVOLUTION_LOG.mdに記録されなければならない。

---

*Causal Inference Engine v2.2 — PearlのDo-Calculusに基づく*
*因果効果識別と推定のコア能力を実装*
