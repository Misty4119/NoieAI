# COUNTERFACTUAL.md

## Counterfactual Reasoning Framework (Counterfactual Reasoning Framework v2.2)

**Definition:** This module is the core sub-module of the NoieLogicAGENTS logic engine, implementing the Counterfactual Reasoning Framework, specializing in handling Layer 3 causal inference (counterfactual) and twin world model computation.

**System Position:** As an L3 detail module of LOGIC_ENGINE.md, specializes in handling the highest level of causal inference — counterfactual reasoning, answering the question "What would have happened if we had made a different choice?"

**Dependency Modules:** LOGIC_ENGINE.md, CAUSAL_INFERENCE.md, ABDUCTIVE_REASONING.md, CONSTRAINTS.md

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

## §1. Theoretical Foundation of Counterfactual Reasoning

### §1.1 Counterfactual Definition

Counterfactual reasoning is the highest level of causal inference, dealing with "what if" type questions.

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                Counterfactual Reasoning Definition                  ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  【Counterfactual Question】                                          ║
║                                                                       ║
║  "If we had done X instead of X', what would Y have been?"       ║
║                                                                       ║
║  Formal expression:                                                  ║
║    P(Y_x = y | X = x', Y = y')                                    ║
║                                                                       ║
║  Interpretation:                                                       ║
║    Given that we observed X = x' and Y = y',                      ║
║    what is the probability that Y would equal y if X were           ║
║    intervened to be x                                              ║
║                                                                       ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  【Counterfactual vs Intervention】                                   ║
║                                                                       ║
║  Intervention:                                                        ║
║    P(Y | do(X = x)) — If we now force X to be x, what happens?  ║
║                                                                       ║
║  Counterfactual:                                                      ║
║    P(Y_x = y | X = x', Y = y') — If we had made a different     ║
║    choice, knowing the current result, what would happen?            ║
║                                                                       ║
║  Key difference:                                                     ║
║    - Intervention is prospective (future)                           ║
║    - Counterfactual is retrospective (past+hypothesis)              ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### §1.2 Position in Three-Layer Causal Inference

```text
╔═══════════════════════════════════════════════════════════════════════╗
║              Three-Layer Causal Inference Framework                 ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  Layer 3 ────────── Counterfactual ────────── Highest Level     ║
║  ══════════════════════════════════════════════════════              ║
║  "What would happen if we had made a different choice?"             ║
║  Tools: Structural equations, Twin world model                      ║
║  Math: P(Y_x | X=x', Y=y')                                      ║
║  Implementation: This module                                         ║
║                                                                       ║
║        ═══════════════════════════════════════                       ║
║                                                                       ║
║  Layer 2 ────────── Intervention ────────── Middle Level        ║
║  "If I forcibly change X, how will Y change?"                 ║
║  Tools: do-calculus, truncation factorization                     ║
║  Math: P(Y | do(X=x))                                            ║
║  Implementation: CAUSAL_INFERENCE.md                                ║
║                                                                       ║
║        ═══════════════════════════════════════                       ║
║                                                                       ║
║  Layer 1 ────────── Association ────────── Base Level             ║
║  "When X is observed, what is the probability of Y?"           ║
║  Tools: Conditional probability, Bayesian inference                 ║
║  Math: P(Y | X)                                                 ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

---

## §2. Twin World Model

### §2.1 Twin World Theory

```python
"""
Twin World Model Implementation
"""

from typing import Dict, List, Optional, Set, Any
from dataclasses import dataclass
from enum import Enum
import copy

@dataclass
class StructuralEquationModel:
    """
    Structural Equation Model
    
    M = (U, V, F)
    - U: Set of exogenous variables
    - V: Set of endogenous variables
    - F: Set of structural equations
    """
    exogenous_vars: Set[str]      # U: External variables
    endogenous_vars: Set[str]     # V: Internal variables
    structural_equations: Dict[str, str]  # F: Structural equations
    
    def evaluate(self, assignments: Dict[str, Any]) -> Dict[str, Any]:
        """
        Evaluate structural equations
        
        Compute all endogenous variable values based on current assignments
        """
        # Topological sort ensures parent nodes are computed first
        evaluated = dict(assignments)
        
        for var in self._topological_order():
            if var in self.endogenous_vars:
                equation = self.structural_equations.get(var, "0")
                # Simplified implementation: framework level
                evaluated[var] = self._evaluate_equation(
                    equation, evaluated
                )
        
        return evaluated
    
    def _topological_order(self) -> List[str]:
        """Get topological sort order"""
        # Simplified implementation
        return list(self.endogenous_vars)
    
    def _evaluate_equation(self, equation: str, context: Dict) -> Any:
        """Evaluate single structural equation"""
        # Framework implementation
        return 0.0


@dataclass
class TwinWorld:
    """
    One of the twin worlds: factual or counterfactual
    
    Each world contains:
    - Structural equation model
    - Assignments of exogenous variables (fixed)
    - Intervention set
    """
    sem: StructuralEquationModel
    exogenous_assignments: Dict[str, Any]  # U values
    interventions: Dict[str, Any]  # do(X=x) interventions
    world_id: str  # "factual" or "counterfactual"
    
    def evaluate(self) -> Dict[str, Any]:
        """
        Evaluate entire world
        
        Apply interventions and compute all endogenous variables
        """
        # Merge interventions into exogenous assignments
        context = dict(self.exogenous_assignments)
        context.update(self.interventions)
        
        return self.sem.evaluate(context)
    
    def get(self, variable: str) -> Any:
        """Get value of specific variable"""
        result = self.evaluate()
        return result.get(variable)


class TwinWorldModel:
    """
    Twin World Model
    
    Contains:
    - Factual world: The actual history that occurred
    - Counterfactual world: The hypothetical history
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
        Setup factual world
        
        Infer U values of exogenous variables from observed results
        """
        # Back-infer exogenous variables: infer U from observed results
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
        Setup counterfactual world
        
        Set intervention do(X=x)
        """
        # Use same exogenous assignments as factual world
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
        Infer exogenous variables from observations
        
        This is the key step in counterfactual reasoning
        """
        # Framework implementation: requires complete forward model
        inferred = {}
        
        # Simplified: assume exogenous variables are observable or inferable
        for var in self.sem.exogenous_vars:
            if var in observations:
                inferred[var] = observations[var]
        
        return inferred
```

### §2.2 ctf-calculus and Counterfactual Reasoning Integration

**ctf-calculus** (Correa & Bareinboim, 2025) extends Do-Calculus to the counterfactual domain, implementing a unified reasoning framework for interventions and counterfactuals. This module integrates ctf-calculus to enhance counterfactual computation capabilities.

```text
╔═══════════════════════════════════════════════════════════════════════╗
║        ctf-calculus: Counterfactual Calculus & Twin World        ║
║                     Model Integration                              ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  【ctf-calculus Core Idea】                                       ║
║                                                                       ║
║  Traditional twin world models require complete structural equation  ║
║  models (SEM) to compute counterfactuals.                        ║
║  ctf-calculus provides a method for counterfactual reasoning       ║
║  without requiring complete SEM.                                  ║
║                                                                       ║
║  【Unified Framework】                                            ║
║                                                                       ║
║  The four rules of ctf-calculus allow us to:                    ║
║                                                                       ║
║  • Identify counterfactual probabilities based on observational data  ║
║  • Reduce dependency on strong assumptions (like deterministic functions)║
║  • Achieve partial identification and sensitivity analysis         ║
║                                                                       ║
║  【Integration with Twin World Model】                            ║
║                                                                       ║
║  1. Identification stage: Use ctf-calculus rules to identify     ║
║     identifiable counterfactuals                                    ║
║  2. Computation stage: For identifiable cases, use twin world model ║
║     for precise computation                                        ║
║  3. Estimation stage: For partial identification, return confidence intervals║
║                                                                       ║
║  【Advantage Comparison】                                         ║
║                                                                       ║
║  ┌─────────────────┬───────────────────┬───────────────────────┐    ║
║  │     Method      │   Twin World      │    ctf-calculus     │    ║
║  ├─────────────────┼───────────────────┼───────────────────────┤    ║
║  │  SEM Dependency│      Complete      │     Partial          │    ║
║  │  Identification│      Exact         │  Exact + Partial     │    ║
║  │  Computation   │       High        │       Variable       │    ║
║  │  Applicability │  Fully known     │  Unknown/Partial      │    ║
║  │                 │    structure      │    structure         │    ║
║  └─────────────────┴───────────────────┴───────────────────────┘    ║
║                                                                       ║
║  【Citation】                                                     ║
║  Correa, J.D., & Bareinboim, E. (2025).                          ║
║  "A Calculus for Counterfactual Reasoning."                        ║
║  ICML 2025.                                                       ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

```python
"""
ctf-calculus Integration Implementation
Enhanced Counterfactual Reasoning Engine
"""

class CTFCalculusCounterfactualEngine:
    """
    Counterfactual engine with ctf-calculus integration
    
    Combines advantages of twin world model and ctf-calculus
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
        Enhanced counterfactual computation
        
        First try ctf-calculus identification, fall back to twin world model if fails
        """
        
        # ═══════════════════════════════════════════════════════════
        # STEP 1: Try ctf-calculus identification
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
        # STEP 2: Fall back to twin world model
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
        Try to use ctf-calculus to identify counterfactual
        
        Returns: (is_identifiable, identified_value, confidence)
        """
        # Simplified implementation: framework level
        
        # Check CF1 condition
        # If (Y ⊥⊥ Z | X, W) holds in G_overline{X}
        # can simplify computation
        
        # This requires complete conditional independence testing
        
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
        Compute counterfactual using twin world model
        """
        # Setup factual world
        self.twin_world_model.setup_factual_world(observations)
        
        # Setup counterfactual world
        self.twin_world_model.setup_counterfactual_world(
            treatment=treatment,
            treatment_value=treatment_counterfactual
        )
        
        # Compute counterfactual result
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
        Compute partial identification bounds
        
        When exact identification is not possible, return confidence intervals
        """
        # Use ctf-calculus partial identification theory
        
        # Get required confounding variables
        confounders = self._identify_confounders(treatment, outcome)
        
        # Compute bounds
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
        """Identify confounding variables"""
        # Use back-door criterion
        # Framework implementation
        return []
    
    def _compute_lower_bound(
        self,
        treatment: str,
        treatment_value: Any,
        outcome: str,
        confounders: List[str]
    ) -> float:
        """Compute lower bound"""
        return 0.0
    
    def _compute_upper_bound(
        self,
        treatment: str,
        treatment_value: Any,
        outcome: str,
        confounders: List[str]
    ) -> float:
        """Compute upper bound"""
        return 1.0
```

---

## §3. Counterfactual Computation

### §3.1 Counterfactual Computation Flow

```text
╔═══════════════════════════════════════════════════════════════════════╗
║               Counterfactual Computation Flow                       ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  Step 1: Construct Factual World                                   ║
║  ───────────────────────────────                                    ║
║  - Construct structural equation model from observed (X=x', Y=y')  ║
║  - Back-infer values of exogenous variables U                       ║
║                                                                       ║
║  Step 2: Set Counterfactual Intervention                            ║
║  ───────────────────────────────                                    ║
║  - Under the same structural equations, apply intervention do(X=x) ║
║  - Keep U values unchanged (same as factual world)                ║
║                                                                       ║
║  Step 3: Compute Counterfactual Result                              ║
║  ───────────────────────────────                                    ║
║  - Solve for Y_x in the counterfactual world                      ║
║  - Obtain counterfactual result                                    ║
║                                                                       ║
║  Step 4: Compute Counterfactual Probability                         ║
║  ───────────────────────────────                                    ║
║  - If exact computation not possible, estimate                     ║
║    P(Y_x = y | X=x', Y=y')                                       ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### §3.2 Counterfactual Computation Implementation

```python
"""
Counterfactual Computation Implementation
"""

@dataclass
class CounterfactualResult:
    """Counterfactual computation result"""
    factual_value: Any           # Factual result Y(x')
    counterfactual_value: Any    # Counterfactual result Y(x)
    probability: float          # Counterfactual probability P(Y_x = y | X=x', Y=y')
    causal_effect: float        # Individual Treatment Effect (ITE)
    assumptions: List[str]      # Identification assumptions
    confidence: float           # Confidence


class CounterfactualEngine:
    """
    Counterfactual computation engine
    """
    
    def __init__(self, sem: StructuralEquationModel):
        self.sem = sem
        self.twin_world_model = TwinWorldModel(sem)
    
    def compute_counterfactual(
        self,
        treatment: str,
        treatment_factual: Any,      # Actually occurred treatment
        treatment_counterfactual: Any, # Hypothetical treatment
        outcome: str,
        observed_outcome: Any,
        observations: Dict[str, Any]
    ) -> CounterfactualResult:
        """
        Compute counterfactual result
        
        Question: "What would the result have been if we had chosen treatment_counterfactual?"
        """
        
        # ═══════════════════════════════════════════════════
        # STEP 1: Construct factual world
        # ═══════════════════════════════════════════════════
        
        self.twin_world_model.setup_factual_world(observations)
        
        factual_value = self.twin_world_model.factual_world.get(outcome)
        
        # ═══════════════════════════════════════════════════
        # STEP 2: Set counterfactual intervention
        # ═══════════════════════════════════════════════════
        
        self.twin_world_model.setup_counterfactual_world(
            treatment=treatment,
            treatment_value=treatment_counterfactual
        )
        
        # ═══════════════════════════════════════════════════
        # STEP 3: Compute counterfactual result
        # ═══════════════════════════════════════════════════
        
        counterfactual_value = (
            self.twin_world_model.counterfactual_world.get(outcome)
        )
        
        # ═══════════════════════════════════════════════════
        # STEP 4: Compute Individual Treatment Effect
        # ═══════════════════════════════════════════════════
        
        ite = counterfactual_value - factual_value if isinstance(
            factual_value, (int, float)
        ) else 0.0
        
        # ═══════════════════════════════════════════════════
        # STEP 5: Estimate counterfactual probability
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
        Estimate counterfactual probability
        
        P(Y_x = y | X=x', Y=y')
        
        Methods:
        1. If structural equations are known, compute exactly
        2. If estimation needed, use sensitivity analysis
        """
        # Framework implementation
        # Actual implementation requires data support
        
        # Simplified: return deterministic estimate
        return 1.0
    
    def compute_ite_distribution(
        self,
        treatment: str,
        outcome: str,
        individual_covariates: Dict[str, Any]
    ) -> List[float]:
        """
        Compute Individual Treatment Effect distribution
        
        Used for heterogeneity analysis
        """
        ite_values = []
        
        # Compute ITE for each individual
        # Framework implementation
        
        return ite_values
```

---

## §4. Counterfactual Causal Inference

### §4.1 Types of Counterfactual Causal Inference

```text
╔═══════════════════════════════════════════════════════════════════════╗
║             Types of Counterfactual Causal Inference              ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  1. Attribution                                                    ║
║     ───────────────────────────────                                  ║
║     "Who should be held responsible?"                              ║
║     Compare: factual result vs counterfactual result                 ║
║                                                                       ║
║  2. Proxy Causation                                                ║
║     ───────────────────────────────                                  ║
║     "Would Y have occurred if X had not?"                         ║
║     Use counterfactual to define causation                         ║
║                                                                       ║
║  3. Outcome Explanation                                           ║
║     ───────────────────────────────                                  ║
║     "Why is the result Y?"                                         ║
║     Generate causal chains leading to the result                     ║
║                                                                       ║
║  4. Expected Loss Estimation                                      ║
║     ───────────────────────────────                                  ║
║     "What is the expected loss from choosing A over B?"            ║
║     Use counterfactual to compute opportunity cost                  ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### §4.2 Counterfactual Causal Inference Implementation

```python
"""
Counterfactual Causal Inference Implementation
"""

@dataclass
class AttributionResult:
    """Attribution result"""
    responsible_variable: str
    attribution_score: float
    counterfactual_comparison: Dict[str, Any]


@dataclass
class ExplanationResult:
    """Outcome explanation"""
    outcome: str
    causal_chain: List[str]
    explanation_confidence: float


class CounterfactualCausalInference:
    """
    Counterfactual causal inference engine
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
        Responsibility attribution
        
        Identify main causes of observed result
        """
        best_cause = None
        best_score = -float('inf')
        
        for cause in potential_causes:
            # Compute counterfactual: what if this cause had not occurred?
            cf_result = self._compute_counterfactual_omission(
                cause, outcome, observations
            )
            
            # Attribution score: difference between counterfactual and observed
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
        Outcome explanation
        
        Generate causal chain explanation leading to the result
        """
        # Backtrack from result to root causes
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
        Estimate expected loss
        
        Calculate opportunity cost of choosing action_taken over action_alternative
        """
        # Compute utility of factual result
        factual_result = self._compute_factual_outcome(
            action_taken, outcome, observations
        )
        factual_utility = utility_function.get(factual_result, 0.0)
        
        # Compute utility of counterfactual result
        cf_result = self._compute_counterfactual_switch(
            action_taken, action_alternative, outcome, observations
        )
        cf_utility = utility_function.get(cf_result.counterfactual_value, 0.0)
        
        # Expected loss
        expected_loss = factual_utility - cf_utility
        
        return expected_loss
    
    def _compute_counterfactual_omission(
        self,
        variable: str,
        outcome: str,
        observations: Dict[str, Any]
    ) -> CounterfactualResult:
        """
        Compute omission counterfactual
        
        "What if this variable had not occurred?"
        """
        # Framework implementation
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
        """Compute attribution score"""
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
        """Backtrack causal chain"""
        chain = [outcome]
        
        # Simplified implementation
        current = outcome
        
        for _ in range(5):  # Maximum 5 steps
            parents = self.causal_graph.get_parents(current)
            if not parents:
                break
            
            # Select most relevant parent node
            # Framework implementation
            break
        
        return chain
    
    def _compute_factual_outcome(
        self,
        action: str,
        outcome: str,
        observations: Dict[str, Any]
    ) -> Any:
        """Compute factual result"""
        return observations.get(outcome, 0.0)
    
    def _compute_counterfactual_switch(
        self,
        action_taken: str,
        action_alternative: str,
        outcome: str,
        observations: Dict[str, Any]
    ) -> CounterfactualResult:
        """Compute switch counterfactual"""
        # Framework implementation
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

## §5. Counterfactual Decision Applications

### §5.1 Counterfactual-assisted Decision Making

```python
"""
Counterfactual-assisted Decision Making
"""

class CounterfactualDecisionSupport:
    """
    Counterfactual decision support system
    
    Uses counterfactual reasoning to assist high-risk decisions
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
        Analyze decision
        
        Use counterfactual reasoning to evaluate decision quality
        """
        results = {
            "decision": decision,
            "analysis": {}
        }
        
        for alt in alternatives:
            if alt == decision:
                continue
            
            # Compute expected loss
            expected_loss = self.causal_inference.estimate_expected_loss(
                decision, alt, outcome, utility_function, observations
            )
            
            # Compute responsibility attribution
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
        What-if analysis
        
        "What would happen if we intervene on X?"
        """
        treatment = list(intervention.keys())[0]
        treatment_value = intervention[treatment]
        
        # Use counterfactual engine to compute
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

## §6. Integration with LOGIC_ENGINE

### §6.1 Integration Interface

```python
"""
Counterfactual Reasoning and Logic Engine Integration
"""

def apply_counterfactual_to_decision(
    decision_input: Dict,
    context: "DecisionContext"
) -> "DecisionOutput":
    """
    Apply counterfactual reasoning to decision flow
    
    Triggered when needing to answer "what if" type questions
    """
    from CAUSAL_INFERENCE import CausalInferenceEngine
    
    # 1. Extract decision-related variables
    decision_var = decision_input.get("decision")
    outcome_var = decision_input.get("outcome")
    alternatives = decision_input.get("alternatives", [])
    
    # 2. Get causal graph
    graph = BuildOrRetrieveCausalGraph(decision_input, context)
    
    # 3. Initialize counterfactual engine
    # Requires structural equation model
    sem = StructuralEquationModel(
        exogenous_vars=set(),
        endogenous_vars=set(),
        structural_equations={}
    )
    
    cf_engine = CounterfactualEngine(sem)
    cf_inference = CounterfactualCausalInference(graph)
    
    # 4. Execute counterfactual analysis
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
    
    # 5. Generate decision recommendation
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

## §7. Dependencies & Constraints

### §7.1 Module Dependencies

| Dependency Module | Description | Reference |
| --- | --- | --- |
| LOGIC_ENGINE.md | Parent module | Call entry |
| CAUSAL_INFERENCE.md | Causal inference engine | do-calculus |
| ABDUCTIVE_REASONING.md | Abductive reasoning | Anomaly explanation |
| CONSTRAINTS.md | Social Authority Levels | SA-L permission validation |
| FORMAL_VERIFIER.md | Formal verification | Logical closure verification |

### §7.2 Constraint Conditions

| Constraint Type | Description | Boundary |
| --- | --- | :--- |
| Structural Equations | Requires complete SEM | Framework implementation |
| Exogenous Variables | Must be inferable or observable | Limited |
| Confidence Threshold | Confidence < 0.3 mark uncertain | confidence ≥ 0.3 |
| Counterfactual Depth | Maximum backtracking depth | ≤ 5 steps |

---

## §8. Version & Evolution

| Version | Date | Change Summary |
| --- | :--- | :--- |
| v2.2 | 2026-03 | Initial version, corresponding to LOGIC_ENGINE.md §2.3 |
| v2.3 | 2026-03 | Integrated ctf-calculus to enhance counterfactual reasoning |

**Evolution Constraint:** Modifications to this module must not violate the immutable core axioms of NoieLogicAGENTS.md. Any evolution proposals must be recorded to EVOLUTION_LOG.md.

---

*Counterfactual Reasoning Framework v2.2 — Twin World Model & Counterfactual Causal Inference*
*Implements core capabilities of Layer 3 causal inference*
