# ABDUCTIVE_REASONING.md

## Abductive Reasoning Module (Abductive Reasoning Module v2.2)

**Definition:** This module is the core sub-module of the NoieLogicAGENTS logic engine, implementing the Abductive Reasoning Framework, specializing in anomaly detection, candidate cause generation, and best explanation selection.

**System Position:** As an L3 detail module of LOGIC_ENGINE.md, specializes in handling Layer 3 cognitive capability — when observed results cannot be explained by the existing causal graph, generate new causal hypotheses.

**Dependency Modules:** LOGIC_ENGINE.md, CAUSAL_INFERENCE.md, CONSTRAINTS.md, FORMAL_VERIFIER.md

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

## §1. Theoretical Foundation of Abductive Reasoning

### §1.1 Abductive Reasoning Definition

Abductive reasoning (Abduction) is the third form of reasoning proposed by Charles Sanders Peirce, alongside deduction and induction.

```text
╔═══════════════════════════════════════════════════════════════════════╗
║               Comparison of Three Reasoning Forms                 ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  【Deductive Reasoning】                                             ║
║  ─────────────────────────────────────                              ║
║  Premise: All A are B, x is A                                       ║
║  Conclusion: x is B                                                  ║
║  Property: Sound reasoning (if premises are true, conclusion must be true)║
║                                                                       ║
║  【Inductive Reasoning】                                             ║
║  ─────────────────────────────────────                              ║
║  Premise: x₁ is A is B, x₂ is A is B, ...                        ║
║  Conclusion: All A are B                                             ║
║  Property: Probabilistic reasoning (conclusion is probable)             ║
║                                                                       ║
║  【Abductive Reasoning】                                             ║
║  ─────────────────────────────────────                              ║
║  Premise: B is observed, rule A → B                                  ║
║  Conclusion: x is probably A                                          ║
║  Property: Best explanation reasoning (conclusion is hypothetical)        ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### §1.2 Role of Abductive Reasoning in Causal Framework

```text
╔═══════════════════════════════════════════════════════════════════════╗
║           Three-Layer Causal Inference & Abductive Reasoning      ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  Layer 3 ────────── Counterfactual ────────── Highest Level   ║
║  "What would happen if we had made a different choice?"           ║
║                                                                       ║
║        ═══════════════════════════════════════                       ║
║                                                                       ║
║  Layer 2 ────────── Intervention ────────── Middle Level     ║
║  "If I forcibly change X, how will Y change?"                     ║
║                                                                       ║
║        ═══════════════════════════════════════                       ║
║                                                                       ║
║  Layer 1 ────────── Association ────────── Base Level      ║
║  "When X is observed, what is the probability of Y?"            ║
║                                                                       ║
║        ═══════════════════════════════════════                       ║
║                                                                       ║
║  【Position of Abductive Reasoning】                                 ║
║  ─────────────────────────────────────                              ║
║  Abductive reasoning is the core of "causal learning":            ║
║  When observations are inconsistent with causal graph predictions,  ║
║  generate new causal hypotheses                                     ║
║  This is the key capability for discovering causal structure from data║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

---

## §2. Anomaly Detection

### §2.1 Anomaly Definition and Types

```python
"""
Anomaly Detection Module
"""

from typing import Dict, List, Optional, Set, Tuple
from dataclasses import dataclass
from enum import Enum
import math

class AnomalyType(Enum):
    """Anomaly type"""
    RESIDUAL = "residual"           # Residual anomaly
    STRUCTURAL = "structural"        # Structural anomaly
    DISTRIBUTIONAL = "distributional" # Distribution anomaly
    TEMPORAL = "temporal"            # Temporal anomaly
    CAUSAL = "causal"               # Causal anomaly

@dataclass
class Anomaly:
    """Anomaly structure"""
    anomaly_type: AnomalyType
    observed_value: float
    predicted_value: float
    residual: float
    severity: float  # 0-1
    timestamp: Optional[str] = None
    context: Optional[Dict] = None

@dataclass
class AnomalyDetectionResult:
    """Anomaly detection result"""
    is_anomaly: bool
    anomaly: Optional[Anomaly]
    detection_method: str
    confidence: float
    threshold_used: float


class AnomalyDetector:
    """
    Anomaly detector
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
        Detect residual anomaly
        
        Using standardized residual:
        z = (observed - predicted) / std_error
        
        If |z| > threshold, then anomaly
        """
        residual = observed - predicted
        z_score = abs(residual / std_error) if std_error > 0 else float('inf')
        
        is_anomaly = z_score > self.residual_threshold
        
        # Calculate severity
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
        Detect distribution anomaly
        
        Using distribution assumption (e.g., normal distribution) to calculate tail probability
        """
        # Simplified implementation: use Z-score
        mean = distribution_params.get("mean", 0)
        std = distribution_params.get("std", 1)
        
        return self.detect_residual_anomaly(value, mean, std)
    
    def detect_temporal_anomaly(
        self,
        time_series: List[float],
        window_size: int = 10
    ) -> List[AnomalyDetectionResult]:
        """
        Detect temporal anomaly
        
        Using sliding window to detect points deviating from trend
        """
        anomalies = []
        
        for i in range(window_size, len(time_series)):
            # Calculate window mean
            window = time_series[i-window_size:i]
            predicted = sum(window) / window_size
            
            # Detect anomaly
            result = self.detect_residual_anomaly(
                time_series[i],
                predicted
            )
            
            if result.is_anomaly and result.anomaly:
                result.anomaly.timestamp = f"t={i}"
            
            anomalies.append(result)
        
        return anomalies
```

### §2.2 Causal Anomaly Detection

```python
"""
Causal Anomaly Detection
"""

class CausalAnomalyDetector:
    """
    Causal anomaly detector
    
    Detects whether observed results deviate from causal graph predictions
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
        Detect causal anomaly
        
        Steps:
        1. Use causal graph to predict result
        2. Compare prediction with observation
        3. If deviation exceeds threshold, mark as anomaly
        """
        # Extract target variable from observation
        target_var = list(observation.keys())[0]
        
        # Check if it's a node in the graph
        if target_var not in graph.nodes:
            return AnomalyDetectionResult(
                is_anomaly=False,
                anomaly=None,
                detection_method="causal_not_applicable",
                confidence=0.0,
                threshold_used=0.0
            )
        
        # Use causal engine to predict
        treatment = list(conditions.keys())[0] if conditions else None
        
        if treatment:
            effect_result = self.causal_engine.estimate_causal_effect(
                X=treatment,
                Y=target_var
            )
            predicted = effect_result.ate
        else:
            # No intervention, use basic prediction
            predicted = 0.0
        
        observed = observation[target_var]
        
        # Detect anomaly
        return self.anomaly_detector.detect_residual_anomaly(
            observed,
            predicted
        )
```

---

## §3. Candidate Cause Generation

### §3.1 Cause Space Definition

```text
╔═══════════════════════════════════════════════════════════════════════╗
║               Candidate Cause Space                               ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  When anomaly is observed, search space for candidate causes:         ║
║                                                                       ║
║  C(anomaly) = { c | anomaly ∈ Consequence(c) }                  ║
║                                                                       ║
║  Where:                                                              ║
║    - c is the candidate cause                                        ║
║    - Consequence(c) is the set of possible consequences of cause c    ║
║                                                                       ║
║  Candidate cause types:                                              ║
║    1. Direct cause: Starting point of edge directly pointing to anomaly node║
║    2. Indirect cause: Node affecting anomaly through causal path      ║
║    3. Root cause: Cause with no other causes (no incoming edges or incoming edges from exogenous variables)║
║    4. Confounding cause: Node affecting anomaly through back-door path║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### §3.2 Candidate Cause Search Algorithm

```python
"""
Candidate Cause Generation
"""

@dataclass
class CandidateCause:
    """Candidate cause structure"""
    variable: str
    cause_type: str  # direct, indirect, root, confounding
    causal_path: List[str]  # Path from cause to anomaly
    probability: float  # Posterior probability
    complexity: float  # Kolmogorov complexity


class CandidateCauseGenerator:
    """
    Candidate cause generator
    """
    
    def __init__(self, graph: "CausalGraph"):
        self.graph = graph
    
    def generate_candidates(
        self,
        anomaly_variable: str,
        max_depth: int = 3
    ) -> List[CandidateCause]:
        """
        Generate candidate cause list
        """
        candidates = []
        
        # 1. Search for direct causes
        direct_causes = self._find_direct_causes(anomaly_variable)
        for cause in direct_causes:
            candidates.append(CandidateCause(
                variable=cause,
                cause_type="direct",
                causal_path=[cause, anomaly_variable],
                probability=0.8,
                complexity=self._estimate_complexity(cause)
            ))
        
        # 2. Search for indirect causes
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
        
        # 3. Identify root causes
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
        
        # 4. Search for confounding causes
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
        """Find direct causes"""
        return list(self.graph.get_parents(target))
    
    def _find_indirect_causes(
        self, 
        target: str, 
        max_depth: int
    ) -> List[str]:
        """Find indirect causes"""
        indirect = []
        visited = {target}
        
        def dfs(node: str, depth: int):
            if depth >= max_depth:
                return
            
            for parent in self.graph.get_parents(node):
                if parent not in visited:
                    visited.add(parent)
                    # Exclude direct causes
                    if parent not in self.graph.get_parents(target):
                        indirect.append(parent)
                    dfs(parent, depth + 1)
        
        dfs(target, 0)
        return indirect
    
    def _find_root_causes(self, target: str) -> List[str]:
        """Find root causes"""
        root_causes = []
        
        # Traverse all paths to target
        for node in self.graph.nodes:
            if node == target:
                continue
            
            # Check if it's a root node (no parent nodes or parent nodes are exogenous variables)
            parents = self.graph.get_parents(node)
            if len(parents) == 0:
                # Check if there's a path to target
                if self._has_path_to(node, target):
                    root_causes.append(node)
        
        return root_causes
    
    def _find_confounding_causes(self, target: str) -> List[str]:
        """Find confounding causes"""
        # Confounding causes: nodes affecting target through back-door path
        # Simplified implementation
        return []
    
    def _has_path_to(self, source: str, target: str) -> bool:
        """Check if there's a path from source to target"""
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
        """Find causal path from source to target"""
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
        """Find back-door path"""
        # Simplified implementation
        return []
    
    def _estimate_complexity(self, variable: str) -> float:
        """
        Estimate Kolmogorov complexity of variable
        
        Simplified: use variable name length and connectivity in graph as proxy
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

## §4. Best Explanation Selection

### §4.1 Kolmogorov Complexity

```text
╔═══════════════════════════════════════════════════════════════════════╗
║               Principle of Parsimony                              ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  【Kolmogorov Complexity Definition】                                ║
║                                                                       ║
║  K(x) = Length of shortest program that generates x             ║
║                                                                       ║
║  In causal reasoning:                                               ║
║    K(explanation) ∝ Description length of explanation + causal path length║
║                                                                       ║
║  【Best Explanation Selection Criterion】                             ║
║                                                                       ║
║  Select candidate cause with minimum K(explanation):               ║
║    Best = argmin_c K(c | anomaly)                                ║
║                                                                       ║
║  This embodies the "Occam's Razor" principle:                     ║
║    "Entities should not be multiplied beyond necessity"            ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### §4.2 Best Explanation Selection Algorithm

```python
"""
Best Explanation Selection
"""

class BestExplanationSelector:
    """
    Best explanation selector
    
    Uses Kolmogorov complexity to select optimal explanation
    """
    
    def __init__(self):
        self.kolmogorov_constant = 1.0  # Normalization constant
    
    def select_best_explanation(
        self,
        candidates: List[CandidateCause],
        observation: Dict
    ) -> CandidateCause:
        """
        Select best explanation
        
        Selection criterion:
        P(cause|anomaly) ∝ P(anomaly|cause) × P(cause) / K(cause)
        
        Where:
        - P(anomaly|cause): Explanatory power of cause for anomaly
        - P(cause): Prior probability of cause
        - K(cause): Kolmogorov complexity of cause
        """
        scored_candidates = []
        
        for candidate in candidates:
            # Compute posterior score
            posterior = self._compute_posterior(candidate, observation)
            
            # Consider complexity
            complexity_penalty = candidate.complexity * self.kolmogorov_constant
            
            # Final score
            score = posterior - complexity_penalty
            
            scored_candidates.append((candidate, score))
        
        # Sort by score
        scored_candidates.sort(key=lambda x: x[1], reverse=True)
        
        # Return highest score
        return scored_candidates[0][0] if scored_candidates else None
    
    def _compute_posterior(
        self,
        candidate: CandidateCause,
        observation: Dict
    ) -> float:
        """
        Compute posterior probability
        
        P(cause|anomaly) ∝ P(anomaly|cause) × P(cause)
        """
        # Prior (based on cause type)
        prior = self._compute_prior(candidate)
        
        # Likelihood (based on causal path)
        likelihood = self._compute_likelihood(candidate, observation)
        
        # Posterior
        posterior = prior * likelihood
        
        return posterior
    
    def _compute_prior(self, candidate: CandidateCause) -> float:
        """Compute prior probability"""
        # Root causes have highest priority
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
        Compute likelihood
        
        Based on strength of causal path
        """
        # Simplified: shorter path means higher likelihood
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
        Rank all candidate causes
        
        Returns: Ordered list of (cause, score)
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

## §5. Abductive Reasoning Flow

### §5.1 Complete Abductive Reasoning Flow

```python
"""
Complete Abductive Reasoning Flow
"""

@dataclass
class AbductiveResult:
    """Abductive reasoning result"""
    status: str  # "anomaly_detected", "no_anomaly", "inconclusive"
    anomaly: Optional[Anomaly]
    best_explanation: Optional[CandidateCause]
    all_candidates: List[CandidateCause]
    ranked_explanations: List[Tuple[CandidateCause, float]]
    graph_update_proposed: bool
    confidence: float


class AbductiveReasoner:
    """
    Abductive reasoner
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
        Execute complete abductive reasoning flow
        
        Steps:
        1. Anomaly detection
        2. If anomaly detected, generate candidate causes
        3. Select best explanation
        4. If confidence sufficient, propose causal graph update
        """
        # ═══════════════════════════════════════════════════
        # STEP 1: Anomaly detection
        # ═══════════════════════════════════════════════════
        
        target_var = list(observation.keys())[0]
        observed_value = observation[target_var]
        
        # Use causal engine to predict
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
        # If no anomaly, return normal
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
        # STEP 2: Generate candidate causes
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
        # STEP 3: Select best explanation
        # ═══════════════════════════════════════════════════
        
        ranked = self.explanation_selector.rank_explanations(
            candidates, observation
        )
        
        best = ranked[0][0] if ranked else None
        
        # ═══════════════════════════════════════════════════
        # STEP 4: Propose causal graph update (if confidence sufficient)
        # ═══════════════════════════════════════════════════
        
        UPDATE_THRESHOLD = 0.8
        
        graph_update_proposed = False
        if best and best.probability > UPDATE_THRESHOLD:
            graph_update_proposed = True
            # Record to audit
            self._log_graph_update_proposal(best, observation)
        
        return AbductiveResult(
            status="anomaly_detected",
            anomaly=anomaly_result.anomaly,
            best_explanation=best,
            all_candidates=candidates,
            ranked_explanations=ranked[:5],  # Top 5
            graph_update_proposed=graph_update_proposed,
            confidence=best.probability if best else 0.3
        )
    
    def _log_graph_update_proposal(
        self,
        cause: CandidateCause,
        observation: Dict
    ):
        """Record causal graph update proposal"""
        # Actual implementation: write to AUDIT_TRAIL
        pass
```

---

## §6. Integration with LOGIC_ENGINE

### §6.1 Integration Interface

```python
"""
Abductive Reasoning and Logic Engine Integration
"""

def apply_abductive_reasoning_to_observation(
    observation: Dict,
    causal_graph: "CausalGraph",
    context: "DecisionContext"
) -> "AbductiveResult":
    """
    Apply abductive reasoning to observation analysis
    
    Triggered when decision engine detects anomaly:
    1. Analyze observation result
    2. Generate candidate causal explanations
    3. Select best explanation
    4. Propose causal graph update
    """
    from CAUSAL_INFERENCE import CausalInferenceEngine
    
    # Initialize causal engine
    engine = CausalInferenceEngine(causal_graph)
    
    # Execute abductive reasoning
    reasoner = AbductiveReasoner(engine, causal_graph)
    result = reasoner.infer(observation)
    
    # Record to audit
    if result.status == "anomaly_detected":
        # Record anomaly detection
        pass
    
    return result
```

---

## §7. Dependencies & Constraints

### §7.1 Module Dependencies

| Dependency Module | Description | Reference |
| --- | --- | --- |
| LOGIC_ENGINE.md | Parent module | Call entry |
| CAUSAL_INFERENCE.md | Causal inference engine | Anomaly prediction |
| CONSTRAINTS.md | Social Authority Levels | SA-L permission validation |
| FORMAL_VERIFIER.md | Formal verification | Hypothesis validation |

### §7.2 Constraint Conditions

| Constraint Type | Description | Boundary |
| --- | --- | :--- |
| Anomaly Threshold | Residual > 2σ marks anomaly | z > 2.0 |
| Update Threshold | Confidence > 0.8 propose update | confidence ≥ 0.8 |
| Cause Depth | Search depth ≤ 3 | max_depth ≤ 3 |
| Candidate Count | Keep maximum 10 candidates | top 10 |

---

## §8. Version & Evolution

| Version | Date | Change Summary |
| --- | :--- | :--- |
| v2.2 | 2026-03 | Initial version, corresponding to LOGIC_ENGINE.md §3 |

**Evolution Constraint:** Modifications to this module must not violate the immutable core axioms of NoieLogicAGENTS.md. Any evolution proposals must be recorded to EVOLUTION_LOG.md.

---

*Abductive Reasoning Module v2.2 — Anomaly Detection, Cause Generation & Best Explanation Selection*
*Implements core capabilities of causal learning and hypothesis generation*
