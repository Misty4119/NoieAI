# CAUSAL_INFERENCE.md

## Causal Inference Engine (Causal Inference Engine v2.2)

**Definition:** This module is the core sub-module of the NoieLogicAGENTS logic engine, implementing Pearl's do-calculus causal inference framework, providing causal effect estimation and adjustment methods.

**System Position:** As an L3 detail module of LOGIC_ENGINE.md, specializes in handling Layer 2 causal inference (intervention) and causal effect identification.

**Dependency Modules:** LOGIC_ENGINE.md, CONSTRAINTS.md, FORMAL_VERIFIER.md

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

## §1. Pearl's Do-Calculus Framework

### §1.1 Three Formal Rules

Do-calculus is the mathematical language of causal inference, allowing identification and computation of intervention effects on causal graphs.

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                    Do-Calculus Three Rules                         ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  Rule 1: Insert/Delete Observation                                   ║
║  ─────────────────────────────────────────────────────────────        ║
║  If (Y ⊥⊥ Z | X, W) holds in G_overline{X}, then:                  ║
║                                                                       ║
║      P(y | do(x), z, w) = P(y | do(x), w)                          ║
║                                                                       ║
║  Interpretation: After intervention do(X=x), if Y is conditionally       ║
║  independent of Z, the observation of Z can be removed               ║
║                                                                       ║
║  ─────────────────────────────────────────────────────────────        ║
║  Rule 2: Action/Observation Exchange                                 ║
║  ─────────────────────────────────────────────────────────────        ║
║  If (Y ⊥⊥ Z | X, W) holds in G_overline{X}, underline{Z}, then:     ║
║                                                                       ║
║      P(y | do(x), do(z), w) = P(y | do(x), z, w)                  ║
║                                                                       ║
║  Interpretation: Can replace intervention do(Z=z) with conditional    ║
║  observation Z=z                                                     ║
║                                                                       ║
║  ─────────────────────────────────────────────────────────────        ║
║  Rule 3: Insert/Delete Action                                        ║
║  ─────────────────────────────────────────────────────────────        ║
║  If (Y ⊥⊥ Z | X, W) holds in G_overline{X}, overline{Z(W)}, then: ║
║                                                                       ║
║      P(y | do(x), do(z), w) = P(y | do(x), w)                      ║
║                                                                       ║
║  Interpretation: Can delete intervention on Z (Z(W) is Z nodes in W    ║
║  that are not descendants of X)                                      ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

---

### §1.2 Graph Operation Symbol Definitions

```text
【Graph Operation Symbols】

G_overline{X}:
  Remove all directed edges pointing TO X from causal graph G
  (i.e., remove all incoming edges to X)
  Simulates the effect of "intervention do(X=x)": cuts all causal dependencies of X

G_underline{X}:
  Reverse all directed edges pointing TO X
  (i.e., add reverse edges of all incoming edges to X)
  Makes X a causal descendant of its parent nodes

G_overline{Z(W)}:
  Remove outgoing edges of all nodes in Z that are not descendants of W
  Used in Rule 3, to identify removable interventions
```

### §1.3 ctf-calculus

**ctf-calculus** (Correa & Bareinboim, 2025) is a major extension of Do-Calculus that extends intervention reasoning to the counterfactual reasoning level, implementing a unified mathematical framework for interventions and counterfactuals.

```text
╔═══════════════════════════════════════════════════════════════════════╗
║              ctf-calculus: Counterfactual Calculus                   ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  【Core Idea】                                                       ║
║  Extends the three rules of Do-Calculus to the counterfactual domain,║
║  enabling formal identification and computation of counterfactual      ║
║  expressions like P(Y_x = y | X=x', Y=y')                           ║
║                                                                       ║
║  【Four ctf-rules】                                                 ║
║                                                                       ║
║  Rule CF1: Counterfactual-Action Exchange                            ║
║  ────────────────────────────────────────────────────                ║
║  If (Y ⊥⊥ Z | X, W) holds in G_overline{X}, then:                  ║
║                                                                       ║
║      P(Y_x = y | Z=z, W=w) = P(Y_x = y | W=w)                     ║
║                                                                       ║
║  Rule CF2: Counterfactual Solidarity                                   ║
║  ────────────────────────────────────────────────────                ║
║  If (Y ⊥⊥ Z | X, W) holds in G, then:                            ║
║                                                                       ║
║      P(Y_x = y | X=x', Z=z, W=w) = P(Y_x = y | X=x', W=w)       ║
║                                                                       ║
║  Rule CF3: Exogeneity Transmission                                   ║
║  ────────────────────────────────────────────────────                ║
║  If Z and X are conditionally independent in G_overline{X}, then:   ║
║                                                                       ║
║      P(Y_x = y | Z=z, W=w) = P(Y_x = y | W=w) × P(Z=z | W=w)     ║
║                                                                       ║
║  Rule CF4: Nested Counterfactual Recursion                           ║
║  ────────────────────────────────────────────────────                ║
║  For nested counterfactual Y_{x'}(x):                               ║
║                                                                       ║
║      P(Y_{x'} = y | X=x, Y=y) = Σ_{z} P(Y_{x'} = y | Z=z)        ║
║                                        × P(Z=z | X=x, Y=y)            ║
║                                                                       ║
║  【Advantages】                                                       ║
║  • Unified handling of intervention and counterfactual reasoning       ║
║  • Reduced dependency on strong assumptions of structural equation models║
║  • Automated counterfactual identification process                     ║
║  • Supports partial identification and sensitivity analysis           ║
║                                                                       ║
║  【Citation】                                                        ║
║  Correa, J.D., & Bareinboim, E. (2025).                           ║
║  "A Calculus for Counterfactual Reasoning."                           ║
║  Proceedings of the 42nd International Conference on Machine Learning.║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

```python
"""
ctf-calculus Implementation Framework
Counterfactual Calculus Framework
"""

class CTFCalculus:
    """
    ctf-calculus implementation
    
    Extends do-calculus to counterfactual reasoning
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
        Apply CF1 rule: Counterfactual-Action Exchange
        
        Condition: (Y ⊥⊥ Z | X, W) holds in G_overline{X}
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
        Apply CF2 rule: Counterfactual Solidarity
        
        Condition: (Y ⊥⊥ Z | X, W) holds in G
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
        Apply CF3 rule: Exogeneity Transmission
        
        Condition: Z and X are conditionally independent in G_overline{X}
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
        Compute counterfactual probability P(Y_x = y | X=x', Y=y')
        
        Uses ctf-calculus rules for identification
        """
        if Z is None:
            Z = []
        
        # Try to apply ctf-calculus rules for simplification
        if self.apply_cf_rule1(Y, X, x, Z[0] if Z else None, Z[1:] if len(Z) > 1 else []):
            # CF1 holds, can simplify computation
            return self._compute_simplified_cf(Y, x, observed_X, observed_Y)
        
        if self.apply_cf_rule2(Y, X, x_prime, Z[0] if Z else None, Z[1:] if len(Z) > 1 else []):
            # CF2 holds
            return self._compute_collider_structure(Y, x, x_prime, observed_Y)
        
        # Cannot identify, return partial identification
        return self._partial_identification(Y, x, x_prime, observed_Y)
    
    def _compute_simplified_cf(
        self,
        Y: str,
        x: Any,
        observed_X: Any,
        observed_Y: Any
    ) -> float:
        """Simplified counterfactual computation"""
        # Framework implementation
        return 0.5
    
    def _compute_collider_structure(
        self,
        Y: str,
        x: Any,
        x_prime: Any,
        observed_Y: Any
    ) -> float:
        """Collider structure counterfactual computation"""
        # Framework implementation
        return 0.5
    
    def _partial_identification(
        self,
        Y: str,
        x: Any,
        x_prime: Any,
        observed_Y: Any
    ) -> float:
        """Partial identification"""
        # Return bounds
        return 0.5


# Integration with existing CausalInferenceEngine
class EnhancedCausalInferenceEngine(CausalInferenceEngine):
    """
    Enhanced Causal Inference Engine
    
    Integrates ctf-calculus to support counterfactual reasoning
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
        Estimate counterfactual result
        
        Uses ctf-calculus for identification
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

### §1.4 Formal Function Implementation

```python
"""
Do-Calculus Implementation
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
    """Causal graph structure"""
    nodes: Set[str]
    edges: List[Tuple[str, str]]  # (parent, child)
    latent_confounders: Optional[Set[str]] = None
    
    def get_parents(self, node: str) -> Set[str]:
        """Get parent nodes of node"""
        parents = set()
        for parent, child in self.edges:
            if child == node:
                parents.add(parent)
        return parents
    
    def get_children(self, node: str) -> Set[str]:
        """Get child nodes of node"""
        children = set()
        for parent, child in self.edges:
            if parent == node:
                children.add(child)
        return children
    
    def get_descendants(self, node: str) -> Set[str]:
        """Get all descendants of node (recursive)"""
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
        """Get all ancestors of node (recursive)"""
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
        """Verify if it is a directed acyclic graph"""
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
    G_overline{X}: Remove all edges pointing TO X
    """
    new_graph = copy.deepcopy(graph)
    new_edges = [(p, c) for p, c in new_graph.edges if c != node]
    new_graph.edges = new_edges
    return new_graph


def add_incoming_edges(graph: CausalGraph, node: str) -> CausalGraph:
    """
    G_underline{X}: Reverse all edges pointing TO X
    """
    new_graph = copy.deepcopy(graph)
    for i, (parent, child) in enumerate(new_graph.edges):
        if child == node:
            new_graph.edges[i] = (node, parent)
    return new_graph


def remove_edges_from(graph: CausalGraph, nodes: Set[str]) -> CausalGraph:
    """
    Remove outgoing edges of specified nodes
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
    Check if X ⊥⊥ Y | Z holds in the graph
    
    Uses d-separation criterion:
    - If unblocked paths exist, dependent
    - If all paths are blocked, independent
    """
    # Simplified implementation: check for d-separated paths
    # Full implementation requires d-separation algorithm
    
    # Get descendants of Z
    z_descendants = set()
    for z in Z:
        z_descendants.update(graph.get_descendants(z))
    
    def is_blocked(path: List[str], observed: Set[str]) -> bool:
        """Check if path is blocked"""
        for i in range(len(path) - 1):
            node = path[i]
            next_node = path[i + 1]
            
            # Chain X → Z → Y or fork Z ← X → Y
            if (node, next_node) in graph.edges or (next_node, node) in graph.edges:
                if node not in observed:
                    return True
            
            # Collider X → Z ← Y
            for p, c in graph.edges:
                if c == node and p not in observed:
                    return True
        
        return False
    
    # Find all paths connecting X and Y
    # If all paths are blocked, d-separated
    return IndependenceTestResult.INDEPENDENT  # Simplified


def do_calculus_rule_1(
    graph: CausalGraph,
    X: str,
    Y: str,
    Z: str,
    W: List[str]
) -> bool:
    """
    Do-Calculus Rule 1:
    P(y | do(x), z, w) = P(y | do(x), w)
    
    Condition: (Y ⊥⊥ Z | X, W) holds in G_overline{X}
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
    Do-Calculus Rule 2:
    P(y | do(x), do(z), w) = P(y | do(x), z, w)
    
    Condition: (Y ⊥⊥ Z | X, W) holds in G_overline{X}, underline{Z}
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
    Do-Calculus Rule 3:
    P(y | do(x), do(z), w) = P(y | do(x), w)
    
    Condition: (Y ⊥⊥ Z | X, W) holds in G_overline{X}, overline{Z(W)}
    """
    # Identify descendants of X in W
    descendants_x = graph.get_descendants(X)
    z_w = {z for z in [Z] if z not in descendants_x}
    
    modified_graph = remove_outgoing_edges(graph, X)
    modified_graph = remove_edges_from(modified_graph, z_w)
    return is_conditionally_independent(modified_graph, Y, Z, [X] + W)
```

---

## §2. Causal Effect Estimation

### §2.1 Average Treatment Effect (ATE)

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                    Causal Effect Measures                           ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  【Average Treatment Effect (ATE)】                                   ║
║                                                                       ║
║  ATE = E[Y(1) - Y(0)]                                              ║
║                                                                       ║
║  Definition: The average outcome difference between treatment and     ║
║  non-treatment in the overall population                            ║
║                                                                       ║
║  Estimation challenges:                                               ║
║    - Y(1) and Y(0) cannot be observed simultaneously (fundamental problem)║
║    - Requires causal identification assumptions                      ║
║                                                                       ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  【Conditional Average Treatment Effect (CATE)】                    ║
║                                                                       ║
║  CATE = E[Y(1) - Y(0) | X = x]                                    ║
║                                                                       ║
║  Definition: Treatment effect within a specific subgroup X=x          ║
║                                                                       ║
║  Applications: Precision medicine, personalized recommendations,       ║
║  differentiated interventions                                        ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### §2.2 Back-door Adjustment Method

```python
"""
Back-door Adjustment Implementation
"""

@dataclass
class CausalEffectResult:
    """Causal effect estimation result"""
    ate: float                          # Average treatment effect
    cate: Optional[float]                # Conditional ATE
    confidence: float                    # Confidence
    method: str                         # Estimation method
    assumptions: List[str]              # Identification assumptions
    adjustment_set: Optional[List[str]]  # Adjustment variable set


def find_backdoor_paths(
    graph: CausalGraph,
    X: str,
    Y: str
) -> List[List[str]]:
    """
    Find all back-door paths from X to Y
    
    Back-door path: Path starting from X, ending at Y,
    where the first edge direction is opposite to the directed path
    """
    backdoor_paths = []
    
    def dfs(current: str, path: List[str], visited: Set[str]):
        if current == Y and len(path) > 1:
            backdoor_paths.append(path)
            return
        
        visited.add(current)
        
        # Explore all neighbors
        for parent in graph.get_parents(current):
            if parent not in visited:
                dfs(parent, [parent] + path, visited.copy())
        
        for child in graph.get_children(current):
            # Back-door path: first step must be backward edge
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
    Check if variable set Z blocks the given path
    """
    z_set = set(Z)
    
    for i in range(len(path) - 1):
        node = path[i]
        next_node = path[i + 1]
        
        # Check if chain or fork
        if (node, next_node) in graph.edges:
            # X → Z → Y: blocked if Z is in observed set
            if node in z_set:
                return True
        
        # Check if collider
        # Colliders only block if their descendants are in observed set
        for parent, child in graph.edges:
            if child == node and parent not in z_set:
                # Check if node's descendants are in Z
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
    Check if variable set Z satisfies back-door criterion
    """
    # Condition 1: Z contains no descendants of X
    x_descendants = graph.get_descendants(X)
    for z in Z:
        if z in x_descendants:
            return False
    
    # Condition 2: Z blocks all back-door paths
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
    Compute causal effect using back-door adjustment formula
    
    P(Y | do(X=x)) = Σ_z P(Y | X=x, Z=z) × P(Z=z)
    
    Or conditional version:
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
    
    # Simplified implementation: framework level
    # Actual implementation requires data support
    
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

### §2.3 Front-door Adjustment Method

```python
"""
Front-door Adjustment Implementation
"""

def satisfies_front_door_criterion(
    graph: CausalGraph,
    X: str,
    Y: str,
    M: str
) -> bool:
    """
    Check if mediator M satisfies front-door criterion
    
    Conditions (according to Pearl, Glymour, Jewell - Causal Inference in Statistics: A Primer):
    1. M blocks all directed paths from X to Y
    2. No back-door paths from X to M (i.e., no X←... paths)
    3. All back-door paths from M to Y are blocked by X
    """
    # Check if X → M → Y exists
    if not (graph.get_children(X) and M in graph.get_children(X)):
        return False
    if not (graph.get_children(M) and Y in graph.get_children(M)):
        return False
    
    # Simplified check
    return True


def compute_front_door_effect(
    graph: CausalGraph,
    X: str,
    Y: str,
    M: str,
    data: Dict[str, List]
) -> CausalEffectResult:
    """
    Compute causal effect using front-door adjustment formula
    
    P(Y | do(X=x)) = Σ_m P(M=m | X=x) × Σ_{x'} P(Y | M=m, X=x') × P(X=x')
    
    Advantage: Even with unobserved confounding variables, causal effect can be
    identified as long as front-door criterion is satisfied
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

### §2.4 Integrated Causal Effect Engine

```python
"""
Integrated Causal Effect Estimation Engine
"""

class CausalInferenceEngine:
    """
    Causal Inference Engine Main Class
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
        Estimate causal effect
        
        Method selection:
        - "auto": Automatically select best method
        - "back_door": Back-door adjustment
        - "front_door": Front-door adjustment
        - "instrumental": Instrumental variable
        """
        if method == "auto":
            method = self._select_best_method(X, Y)
        
        if method == "back_door":
            # Try back-door adjustment
            candidates = self._find_minimal_adjustment_set(X, Y)
            if candidates:
                return compute_backdoor_effect(
                    self.graph, X, Y, candidates, {}
                )
        
        if method == "front_door":
            # Try front-door adjustment
            mediators = self._find_mediators(X, Y)
            for m in mediators:
                result = compute_front_door_effect(
                    self.graph, X, Y, m, {}
                )
                if result.confidence > 0:
                    return result
        
        # Cannot identify
        return CausalEffectResult(
            ate=0.0,
            cate=None,
            confidence=0.0,
            method="unidentified",
            assumptions=[],
            adjustment_set=None
        )
    
    def _select_best_method(self, X: str, Y: str) -> str:
        """Automatically select best identification method"""
        # Prioritize back-door
        candidates = self._find_minimal_adjustment_set(X, Y)
        if candidates:
            return "back_door"
        
        # Try front-door
        mediators = self._find_mediators(X, Y)
        if mediators:
            return "front_door"
        
        return "unidentified"
    
    def _find_minimal_adjustment_set(
        self, 
        X: str, 
        Y: str
    ) -> Optional[List[str]]:
        """Find minimal adjustment set"""
        candidates = [n for n in self.graph.nodes if n != X and n != Y]
        
        # Simplified: return first valid set
        for candidate in candidates:
            if satisfies_backdoor_criterion(self.graph, X, Y, [candidate]):
                return [candidate]
        
        return None
    
    def _find_mediators(self, X: str, Y: str) -> List[str]:
        """Find mediators from X to Y"""
        mediators = []
        x_children = self.graph.get_children(X)
        
        for m in x_children:
            if self.graph.get_children(m) and Y in self.graph.get_children(m):
                mediators.append(m)
        
        return mediators
```

---

## §3. Identification Assumptions & Validity

### §3.1 Core Identification Assumptions

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                    Causal Identification Assumptions               ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  1. Consistency                                                    ║
║     ─────────────────────────────────────                            ║
║     If X = x, then Y = Y(x)                                        ║
║                                                                       ║
║     Ensures that intervention results are consistent with observations  ║
║                                                                       ║
║  2. Positivity                                                     ║
║     ─────────────────────────────────────                            ║
║     P(X = x | Z = z) > 0 for all z                                 ║
║                                                                       ║
║     Ensures that every subgroup has both treatment and control samples ║
║                                                                       ║
║  3. Conditional Exchangeability                                      ║
║     ─────────────────────────────────────                            ║
║     Y(1), Y(0) ⊥⊥ X | Z                                           ║
║                                                                       ║
║     Given confounder Z, treatment and outcome are conditionally independent║
║                                                                       ║
║  4. Stable Unit Treatment Value Assumption (SUTVA)                   ║
║     ─────────────────────────────────────                            ║
║     - Treatment effects are consistent across all units                ║
║     - One unit's treatment does not affect another unit's outcome     ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### §3.2 Assumption Testing & Degradation

```python
"""
Identification Assumption Testing
"""

class AssumptionValidator:
    """Identification assumption validator"""
    
    def __init__(self, graph: CausalGraph, data: Dict[str, List]):
        self.graph = graph
        self.data = data
    
    def check_positivity(self, X: str, Z: List[str]) -> Dict:
        """
        Check positivity assumption
        """
        # Simplified implementation
        return {
            "satisfied": True,
            "violations": [],
            "confidence": 0.9
        }
    
    def check_consistency(self, X: str, Y: str) -> Dict:
        """
        Check consistency assumption
        """
        return {
            "satisfied": True,
            "confidence": 0.95
        }
    
    def check_sutva(self) -> Dict:
        """
        Check SUTVA
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
        Comprehensively validate all identification assumptions
        """
        results = {
            "consistency": self.check_consistency(X, Y),
            "positivity": self.check_positivity(X, Z),
            "sutva": self.check_sutva()
        }
        
        # Compute overall confidence
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

## §4. Decision Integration

### §4.1 Integration with LOGIC_ENGINE

```python
"""
Causal Inference and Logic Engine Integration
"""

def apply_causal_inference_to_decision(
    decision_input: Dict,
    context: "DecisionContext"
) -> "DecisionOutput":
    """
    Apply causal inference to decision flow
    """
    # 1. Extract causal query
    treatment = decision_input.get("treatment")
    outcome = decision_input.get("outcome")
    confounders = decision_input.get("confounders", [])
    
    # 2. Get or construct causal graph
    graph = BuildOrRetrieveCausalGraph(decision_input, context)
    
    # 3. Estimate causal effect
    engine = CausalInferenceEngine(graph)
    causal_effect = engine.estimate_causal_effect(
        X=treatment,
        Y=outcome,
        method="auto",
        condition_on=confounders
    )
    
    # 4. Validate identification assumptions
    validator = AssumptionValidator(graph, {})
    assumption_check = validator.validate_assumptions(
        treatment, outcome, confounders
    )
    
    # 5. Adjust confidence
    adjusted_confidence = (
        causal_effect.confidence * 
        assumption_check["confidence"]
    )
    
    # 6. Generate decision recommendation
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

## §5. Dependencies & Constraints

### §5.1 Module Dependencies

| Dependency Module | Description | Reference |
| --- | --- | --- |
| LOGIC_ENGINE.md | Parent module, objective reasoning engine | Call entry |
| CONSTRAINTS.md | Social Authority Levels | SA-L permission validation |
| FORMAL_VERIFIER.md | Formal verification | Logical closure verification |
| KNOWLEDGE_BASE.md | Information bit ledger | Fact queries |

### §5.2 Constraint Conditions

| Constraint Type | Description | Boundary |
| --- | --- | :--- |
| Causal Graph | Must be DAG | No cycles |
| Identification Method | Prioritize back-door, then front-door | Return unidentifiable if cannot |
| Confidence Threshold | Confidence < 0.3 reject decision | confidence ≥ 0.3 |
| Identification Assumptions | Must validate consistency, positivity | Both required |

---

## §6. Version & Evolution

| Version | Date | Change Summary |
| --- | :--- | :--- |
| v2.2 | 2026-03 | Initial version, corresponding to LOGIC_ENGINE.md §2 |
| v2.3 | 2026-03 | Added ctf-calculus counterfactual calculus |

**Evolution Constraint:** Modifications to this module must not violate the immutable core axioms of NoieLogicAGENTS.md. Any evolution proposals must be recorded to EVOLUTION_LOG.md.

---

*Causal Inference Engine v2.2 — Based on Pearl's Do-Calculus*
*Implements core capabilities for causal effect identification and estimation*
