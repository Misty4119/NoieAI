# NoieLogicAGENTS — Logical Closure Detector

**Version:** Logic-OS v2.2  
**Module Code:** CLOSURE_DETECTOR  
**Responsibility:** Logical closure computation, missing step identification, closure completeness evaluation  
**Parent Module:** FORMAL_VERIFIER

---

> ⚠️ Critical Safety & Decision Protocol (CRITICAL SAFETY & DECISION PROTOCOL v2.2):
> 1. Strictly adhere to CONSTRAINTS.md and Social Authority levels (SA-L0 to SA-L5).
> 2. Causal inference: All decisions must be based on causal graphs (DAGs), with causal mechanisms annotated.
> 3. Subject-object separation: Decision inference must not confuse self-state with environmental state.
> 4. Formal verification: High-risk decision paths must pass logical closure verification.
> 5. Shadow simulation: For SA-L3+ operations, first rehearse consequences in SANDBOX.
> 6. Information bit integrity: Never fabricate information bits. If KNOWLEDGE_BASE is empty, explicitly state "DATA_MISSING".
> 7. Cognitive resource constraints: Decision depth must not exceed available cognitive resources.
> 8. Audit: Record all conflicts, rejections, and formal verification results to AUDIT_TRAIL.
> 9. Survival priority: All decisions must be verified to not lead to absorbing states before execution.
> 10. Self-evolution: When the axiom system evolves, immutable core must be preserved.

---

## §0. Logical Closure Framework

### §0.1 Core Definitions

```text
【Logical Closure Definition】

Given a set of propositions S, the logical closure CL(S) is defined as:

  CL(S) = { φ | S ⊢ φ }

That is: the set of all propositions that can be logically derived from S.

Formal properties:

  1. Inclusion: S ⊆ CL(S)
  2. Idempotence: CL(CL(S)) = CL(S)
  3. Monotonicity: If S ⊆ T, then CL(S) ⊆ CL(T)

【Closure Operator】

  Closure(P) = S₀ ∪ S₁ ∪ S₂ ∪ ...

  Where:
    S₀ = P (proposition set)
    Sᵢ₊₁ = Sᵢ ∪ { φ | ∃r ∈ R, ∃s ∈ Sᵢ: r(s) = φ }

  Iterate until Sᵢ₊₁ = Sᵢ (fixed point)
```

### §0.2 Closure Completeness

```text
【Closure Completeness Definition】

A proof chain P is closure-complete if and only if:

  Closure(P) ⊆ P

That is: all conclusions that can be logically derived from P are already explicitly stated in P.

【Missing Steps】

Missing steps are:
  φ ∈ Closure(P) and φ ∉ P

These are inference steps that are logically required but not explicitly stated.
```

---

## §1. Logical Closure Computation

### §1.1 Closure Computation Engine

```python
class LogicalClosureEngine:
    """
    Logical Closure Computation Engine
    
    Compute the logical closure of proposition sets
    """
    
    def __init__(self):
        self.inference_rules = InferenceRuleRegistry()
        self.axiom_set = AxiomSet()
        self.max_iterations = 1000  # Prevent infinite loops
        self.closure_cache = {}
    
    def compute_closure(self, proposition_set: Set[Proposition]) -> Set[Proposition]:
        """
        Compute the closure of a proposition set
        
        Method: Iteratively apply inference rules until fixed point
        
        Returns:
            Closure set (Closure Set)
        """
        
        # Cache check
        cache_key = self._hash_proposition_set(proposition_set)
        if cache_key in self.closure_cache:
            return self.closure_cache[cache_key]
        
        # Initialize
        current_closure = set(proposition_set)
        previous_closure = set()
        
        iteration = 0
        
        while current_closure != previous_closure and iteration < self.max_iterations:
            iteration += 1
            previous_closure = set(current_closure)
            
            # Apply all inference rules
            new_propositions = self._apply_inference_rules(current_closure)
            
            # Add new propositions
            current_closure.update(new_propositions)
        
        if iteration >= self.max_iterations:
            # Log warning: possible infinite loop
            self._log_iteration_warning(proposition_set, iteration)
        
        # Cache result
        self.closure_cache[cache_key] = current_closure
        
        return current_closure
    
    def _apply_inference_rules(self, propositions: Set[Proposition]) -> Set[Proposition]:
        """
        Apply inference rules to generate new propositions
        """
        
        new_propositions = set()
        
        # Iterate through all proposition pairs
        proposition_list = list(propositions)
        
        for i, prop_i in enumerate(proposition_list):
            # Unary inference (Unary rules)
            unary_results = self._apply_unary_rules(prop_i)
            new_propositions.update(unary_results)
            
            # Binary inference (Binary rules)
            for prop_j in proposition_list[i+1:]:
                binary_results = self._apply_binary_rules(prop_i, prop_j)
                new_propositions.update(binary_results)
        
        # Filter out already existing propositions
        new_propositions = new_propositions - propositions
        
        return new_propositions
    
    def _apply_unary_rules(self, proposition: Proposition) -> Set[Proposition]:
        """
        Apply unary inference rules
        
        For example:
        - Negation elimination: ¬¬P → P
        - Double negation introduction: P → ¬¬P
        """
        
        results = set()
        
        # Rule 1: Negation elimination
        if proposition.is_double_negation:
            results.add(proposition.eliminate_double_negation())
        
        # Rule 2: Conjunction elimination
        if proposition.is_conjunction:
            results.add(proposition.conjunct_1)
            results.add(proposition.conjunct_2)
        
        # Rule 3: Conditional transmission
        if proposition.is_conditional:
            # P → Q can lead to ¬Q → ¬P (contrapositive)
            results.add(Proposition.contrapositive(proposition))
        
        return results
    
    def _apply_binary_rules(self, prop_a: Proposition, prop_b: Proposition) -> Set[Proposition]:
        """
        Apply binary inference rules
        
        For example:
        - Modus ponens: (P → Q), P ⊢ Q
        - Disjunctive syllogism: (P ∨ Q), ¬P ⊢ Q
        - Hypothetical syllogism: (P → Q), (Q → R) ⊢ (P → R)
        """
        
        results = set()
        
        # Rule: Modus Ponens
        if prop_a.is_conditional and prop_b.entails(prop_a.antecedent):
            results.add(prop_a.consequent)
        
        # Rule: Modus Tollens
        if prop_a.is_conditional and prop_b.entails(Proposition.negate(prop_a.consequent)):
            results.add(Proposition.negate(prop_a.antecedent))
        
        # Rule: Hypothetical Syllogism
        if prop_a.is_conditional and prop_b.is_conditional:
            if prop_a.consequent.entails(prop_b.antecedent):
                results.add(Proposition.conditional(prop_a.antecedent, prop_b.consequent))
        
        # Rule: Disjunctive Syllogism
        if prop_a.is_disjunction:
            if prop_b.entails(Proposition.negate(prop_a.disjunct_1)):
                results.add(prop_a.disjunct_2)
            if prop_b.entails(Proposition.negate(prop_a.disjunct_2)):
                results.add(prop_a.disjunct_1)
        
        # Rule: Conjunction Introduction
        results.add(Proposition.conjunction(prop_a, prop_b))
        
        # Rule: Disjunction Introduction
        results.add(Proposition.disjunction(prop_a, prop_b))
        results.add(Proposition.disjunction(prop_b, prop_a))
        
        return results
```

### §1.2 Incremental Closure Computation

```python
class IncrementalClosureEngine:
    """
    Incremental Closure Computation Engine
    
    Incrementally update closure when proposition set changes
    """
    
    def __init__(self):
        self.base_closure = None
        self.base_set = None
        self.incremental_engine = LogicalClosureEngine()
    
    def compute_incremental(self, original_set: Set[Proposition], 
                           added: Set[Proposition],
                           removed: Set[Proposition]) -> Set[Proposition]:
        """
        Incrementally compute closure
        
        Strategy:
        1. If only additions: Compute closure of added propositions, merge with original closure
        2. If only removals: Need to recompute (because closure does not support removal)
        3. If both: Process additions first, then recompute
        """
        
        if removed and not added:
            # Need full recomputation
            new_set = original_set - removed
            return self.incremental_engine.compute_closure(new_set)
        
        if added and not removed:
            # Incremental update
            # Strategy: Compute incremental closure of newly added propositions relative to original closure
            
            if self.base_closure is None:
                self.base_closure = self.incremental_engine.compute_closure(original_set)
                self.base_set = original_set
            
            # Closure of new propositions relative to original closure
            combined = self.base_closure | added
            incremental_closure = self.incremental_engine.compute_closure(combined)
            
            # Incremental = new closure - original closure (excluding newly added propositions)
            incremental = incremental_closure - self.base_closure
            
            return self.base_closure | incremental
        
        # Both have changes
        new_set = (original_set | added) - removed
        return self.incremental_engine.compute_closure(new_set)
    
    def invalidate_cache(self):
        """
        Invalidate cache
        
        Call when inference rules or axioms change
        """
        
        self.base_closure = None
        self.base_set = None
```

---

## §2. Missing Step Identification

### §2.1 Missing Step Detector

```python
class MissingStepDetector:
    """
    Missing Step Detector
    
    Identify missing logical steps in proof chains
    """
    
    def __init__(self):
        self.closure_engine = LogicalClosureEngine()
    
    def detect_missing_steps(self, proof_chain: ProofChain) -> MissingStepResult:
        """
        Detect missing steps
        
        Steps:
        1. Compute closure of proof chain proposition set
        2. Identify propositions in closure but not in original proof
        3. For each missing proposition, try to identify required inference
        """
        
        # Step 1: Extract proposition set
        proposition_set = set(step.conclusion for step in proof_chain.steps)
        
        # Step 2: Compute closure
        closure = self.closure_engine.compute_closure(proposition_set)
        
        # Step 3: Identify missing propositions
        missing_propositions = closure - proposition_set
        
        # Step 4: Analyze each missing proposition
        missing_steps = []
        
        for missing_prop in missing_propositions:
            # Try to find inference path from existing propositions to missing proposition
            inference_path = self._find_inference_path(
                proposition_set, 
                missing_prop
            )
            
            if inference_path:
                missing_steps.append(MissingStep(
                    proposition=missing_prop,
                    from_premises=inference_path.premises,
                    via_rule=inference_path.rule,
                    confidence=inference_path.confidence,
                    severity=self._calculate_severity(inference_path)
                ))
            else:
                # Cannot automatically identify, requires human intervention
                missing_steps.append(MissingStep(
                    proposition=missing_prop,
                    from_premises=None,
                    via_rule=None,
                    confidence=0.0,
                    severity='HIGH',
                    requires_manual=True
                ))
        
        return MissingStepResult(
            missing_steps=missing_steps,
            total_missing=len(missing_steps),
            closure_size=len(closure),
            proof_size=len(proposition_set),
            completeness_ratio=len(proposition_set) / len(closure) if closure else 0.0
        )
    
    def _find_inference_path(self, premises: Set[Proposition], 
                            target: Proposition) -> InferencePath:
        """
        Find inference path from premises to target proposition
        """
        
        # Use bidirectional search
        # Forward: Search from premises toward target
        # Backward: Search from target toward premises
        
        forward_frontier = set(premises)
        forward_visited = set(premises)
        forward_paths = {p: [p] for p in premises}
        
        backward_frontier = {target}
        backward_visited = {target}
        backward_paths = {target: [target]}
        
        max_depth = 10
        
        for depth in range(max_depth):
            # Expand forward
            new_forward = set()
            for prop in forward_frontier:
                # Apply forward inference rules
                results = self._apply_forward_rules(prop)
                for result in results:
                    if result not in forward_visited:
                        forward_visited.add(result)
                        new_forward.add(result)
                        forward_paths[result] = forward_paths[prop] + [result]
            
            # Check if target found
            if target in new_forward:
                return InferencePath(
                    premises=premises,
                    target=target,
                    path=forward_paths[target],
                    rule='composite',
                    confidence=0.8 ** depth
                )
            
            forward_frontier = new_forward
            
            # Expand backward
            new_backward = set()
            for prop in backward_frontier:
                # Apply backward inference rules
                requirements = self._apply_backward_rules(prop)
                for req in requirements:
                    if req not in backward_visited:
                        backward_visited.add(req)
                        new_backward.add(req)
                        backward_paths[req] = [req] + backward_paths[prop]
            
            backward_frontier = new_backward
            
            # Check for meeting point
            intersection = forward_visited & backward_visited
            if intersection:
                meeting_point = intersection.pop()
                combined_path = forward_paths[meeting_point] + backward_paths[meeting_point][1:]
                return InferencePath(
                    premises=premises,
                    target=target,
                    path=combined_path,
                    rule='bidirectional',
                    confidence=0.7 ** depth
                )
        
        return None
    
    def _apply_forward_rules(self, proposition: Proposition) -> Set[Proposition]:
        """
        Apply forward inference rules
        """
        
        results = set()
        
        # Conjunction elimination
        if proposition.is_conjunction:
            results.add(proposition.conjunct_1)
            results.add(proposition.conjunct_2)
        
        # Conditional antecedent
        # This requires more complex implementation
        
        return results
    
    def _apply_backward_rules(self, proposition: Proposition) -> Set[Proposition]:
        """
        Apply backward inference rules
        """
        
        results = set()
        
        # Conjunction introduction: requires two premises
        if proposition.is_conjunction:
            results.add(proposition.conjunct_1)
            results.add(proposition.conjunct_2)
        
        # Disjunction elimination: requires case analysis
        if proposition.is_disjunction:
            # Return empty set, as more information needed
            pass
        
        return results
    
    def _calculate_severity(self, inference_path: InferencePath) -> str:
        """
        Calculate severity of missing step
        """
        
        if inference_path.confidence >= 0.9:
            return 'LOW'
        elif inference_path.confidence >= 0.7:
            return 'MEDIUM'
        elif inference_path.confidence >= 0.5:
            return 'HIGH'
        else:
            return 'CRITICAL'
```

### §2.2 Automatic Completion

```python
class ProofCompleter:
    """
    Proof Automatic Completer
    
    Automatically fill in missing inference steps
    """
    
    def __init__(self):
        self.missing_step_detector = MissingStepDetector()
        self.inference_engine = LogicalClosureEngine()
    
    def complete_proof(self, proof_chain: ProofChain) -> ProofChain:
        """
        Automatically complete proof chain
        """
        
        # Detect missing steps
        missing_result = self.missing_step_detector.detect_missing_steps(proof_chain)
        
        # Copy original proof
        completed = ProofChain(
            id=proof_chain.id,
            target=proof_chain.target,
            nodes=list(proof_chain.nodes),
            edges=list(proof_chain.edges)
        )
        
        # Add non-critical missing steps
        auto_completed_count = 0
        
        for missing in missing_result.missing_steps:
            if missing.severity in ['LOW', 'MEDIUM'] and not missing.requires_manual:
                # Automatically fill in
                new_step = ProofStep(
                    id=f"auto_{missing.id}",
                    conclusion=missing.proposition,
                    premises=list(missing.from_premises),
                    source='auto_inference',
                    rule=missing.via_rule,
                    confidence=missing.confidence,
                    is_auto_generated=True
                )
                
                completed.steps.append(new_step)
                auto_completed_count += 1
        
        # Log completion result
        self._log_completion(proof_chain.id, auto_completed_count, 
                           missing_result.total_missing)
        
        return completed
    
    def _log_completion(self, proof_id, completed_count, total_missing):
        """
        Log completion
        """
        
        AppendToAuditTrail({
            'event': 'PROOF_COMPLETION',
            'proof_id': proof_id,
            'auto_completed': completed_count,
            'total_missing': total_missing,
            'remaining_missing': total_missing - completed_count
        })
```

---

## §3. Closure Completeness Evaluation

### §3.1 Completeness Metrics

```python
class ClosureCompletenessEvaluator:
    """
    Closure Completeness Evaluator
    
    Evaluate closure completeness of proof chains
    """
    
    def evaluate(self, proof_chain: ProofChain) -> ClosureCompletenessResult:
        """
        Evaluate closure completeness
        """
        
        # Compute proposition set
        proposition_set = set(step.conclusion for step in proof_chain.steps)
        
        # Compute closure
        closure_engine = LogicalClosureEngine()
        closure = closure_engine.compute_closure(proposition_set)
        
        # Compute metrics
        missing = closure - proposition_set
        
        # Completeness ratio
        completeness_ratio = len(proposition_set) / len(closure) if closure else 0.0
        
        # Closure density
        closure_density = len(closure) / (len(proposition_set) ** 2) if proposition_set else 0.0
        
        # Identify type distribution in closure
        type_distribution = self._analyze_closure_types(closure, proposition_set)
        
        return ClosureCompletenessResult(
            proof_id=proof_chain.id,
            
            # Basic metrics
            proposition_count=len(proposition_set),
            closure_count=len(closure),
            missing_count=len(missing),
            
            # Completeness score
            completeness_ratio=completeness_ratio,
            completeness_score=self._score_completeness(completeness_ratio),
            
            # Structural metrics
            closure_density=closure_density,
            
            # Type distribution
            explicit_ratio=type_distribution['explicit'],
            implicit_ratio=type_distribution['implicit'],
            
            # Missing analysis
            missing_propositions=list(missing),
            missing_by_severity=self._categorize_by_severity(proof_chain, missing),
            
            # Recommendations
            recommendations=self._generate_recommendations(completeness_ratio, missing)
        )
    
    def _score_completeness(self, ratio: float) -> float:
        """
        Score based on completeness ratio
        """
        
        if ratio >= 0.95:
            return 1.0
        elif ratio >= 0.80:
            return 0.8
        elif ratio >= 0.60:
            return 0.6
        elif ratio >= 0.40:
            return 0.4
        else:
            return 0.2
    
    def _analyze_closure_types(self, closure: Set[Proposition], 
                              explicit: Set[Proposition]) -> dict:
        """
        Analyze proposition types in closure
        """
        
        explicit_count = len(explicit)
        total_count = len(closure)
        
        return {
            'explicit': explicit_count,
            'implicit': total_count - explicit_count,
            'explicit_ratio': explicit_count / total_count if total_count else 0,
            'implicit_ratio': (total_count - explicit_count) / total_count if total_count else 0
        }
    
    def _categorize_by_severity(self, proof_chain: ProofChain, 
                                missing: Set[Proposition]) -> dict:
        """
        Categorize missing propositions by severity
        """
        
        detector = MissingStepDetector()
        missing_result = detector.detect_missing_steps(proof_chain)
        
        by_severity = {
            'LOW': [],
            'MEDIUM': [],
            'HIGH': [],
            'CRITICAL': []
        }
        
        for missing_step in missing_result.missing_steps:
            by_severity[missing_step.severity].append(missing_step)
        
        return by_severity
    
    def _generate_recommendations(self, completeness_ratio: float, 
                                  missing: Set[Proposition]) -> List[str]:
        """
        Generate improvement recommendations
        """
        
        recommendations = []
        
        if completeness_ratio < 0.5:
            recommendations.append('CRITICAL: Proof chain has significant gaps. Manual review required.')
        elif completeness_ratio < 0.8:
            recommendations.append('HIGH: Proof is incomplete. Consider adding missing inference steps.')
        elif completeness_ratio < 0.95:
            recommendations.append('MEDIUM: Proof has minor gaps. Auto-completion may help.')
        else:
            recommendations.append('LOW: Proof is essentially complete.')
        
        if len(missing) > 10:
            recommendations.append('Large number of implicit conclusions. Consider restructuring proof.')
        
        return recommendations
```

### §3.2 Completeness Report Generation

```python
def generate_closure_completeness_report(proof_chain: ProofChain) -> str:
    """
    Generate closure completeness report
    """
    
    evaluator = ClosureCompletenessEvaluator()
    result = evaluator.evaluate(proof_chain)
    
    report = f"""
================================================================================
                    Logical Closure Completeness Report
================================================================================

Proof ID: {result.proof_id}

--------------------------------------------------------------------------------
Basic Metrics
--------------------------------------------------------------------------------
Proposition count:     {result.proposition_count}
Closure size:          {result.closure_count}
Missing count:         {result.missing_count}

--------------------------------------------------------------------------------
Completeness Score
--------------------------------------------------------------------------------
Completeness ratio:    {result.completeness_ratio:.2%}
Completeness score:    {result.completeness_score:.2f}

Structural metrics:
- Closure density: {result.closure_density:.4f}

Type distribution:
- Explicit propositions: {result.explicit_ratio:.2%}
- Implicit propositions: {result.implicit_ratio:.2%}

--------------------------------------------------------------------------------
Missing Analysis
--------------------------------------------------------------------------------
"""
    
    for severity, steps in result.missing_by_severity.items():
        if steps:
            report += f"\n{severity} severity ({len(steps)} items):\n"
            for step in steps[:5]:  # Display at most 5 items
                report += f"  - {step.proposition}\n"
            if len(steps) > 5:
                report += f"  ... and {len(steps) - 5} more\n"
    
    report += """
--------------------------------------------------------------------------------
Recommendations
--------------------------------------------------------------------------------
"""
    
    for rec in result.recommendations:
        report += f"- {rec}\n"
    
    report += """
================================================================================
"""
    
    return report
```

---

## §4. Audit and Logging

### §4.1 Closure Detection Log

```python
def LogClosureDetectionToAuditTrail(detection_result, proof_chain):
    """
    Record closure detection results to AUDIT_TRAIL
    """
    
    audit_entry = {
        'event': 'CLOSURE_DETECTION',
        'timestamp': GetCurrentTimestamp(),
        
        # Proof identification
        'proof_id': proof_chain.id,
        
        # Detection results
        'completeness_ratio': detection_result.completeness_ratio,
        'completeness_score': detection_result.completeness_score,
        
        # Quantitative metrics
        'proposition_count': detection_result.proposition_count,
        'closure_count': detection_result.closure_count,
        'missing_count': detection_result.missing_count,
        
        # Missing classification
        'missing_low': len(detection_result.missing_by_severity.get('LOW', [])),
        'missing_medium': len(detection_result.missing_by_severity.get('MEDIUM', [])),
        'missing_high': len(detection_result.missing_by_severity.get('HIGH', [])),
        'missing_critical': len(detection_result.missing_by_severity.get('CRITICAL', [])),
        
        # Structural metrics
        'closure_density': detection_result.closure_density,
        
        # Recommendations
        'recommendations': detection_result.recommendations
    }
    
    AppendToAuditTrail(audit_entry)
    
    return audit_entry
```

---

## §5. Version and Evolution

| Version | Date | Change Summary |
|---------|------|---------------|
| v2.2 | 2026-03 | Initial version, established logical closure detection framework |

---

*NoieLogicAGENTS — CLOSURE_DETECTOR Module*  
*Logic-OS v2.2 Logical Closure Detection Core*  
*Compute closure, identify gaps, evaluate completeness*
