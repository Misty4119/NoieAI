# DIVERGENCE_DETECTOR.md

## L2 - Ontological Divergence Detection Engine (with Semantic Collapse Detection)

> **⚠️ CRITICAL SAFETY & TRUTH PROTOCOL:** This module is responsible for detecting all types of ontological divergence (replacing the traditional "hallucination" concept), and includes semantic collapse detection functionality.

---

## 1. Ontological Divergence Taxonomy

### 1.1 Divergence Type Definitions

| Divergence Type | Definition | Detection Mechanism |
|-----------------|-----------|-------------------|
| **Fabrication** | Claims mapping to no object in the information manifold | Knowledge base cross-reference |
| **Provenance Forgery** | Constructing non-existent provenance chains | Source chain cryptographic verification |
| **Confidence Divergence** | Systematic divergence between confidence vector and actual accuracy | Confidence calibration audit |
| **Semantic Drift** | Unexpected deviation in semantic fidelity during morphism mapping | Semantic fidelity comparison |
| **Temporal Misfit** | Facts from different system state cycles mixed into same context | Intrinsic clock consistency verification |
| **Apophenia** | Extracting non-existent information manifold structure from noise | Statistical significance test |
| **Confabulation** | Constructing reasonable justification chain for wrong conclusion | Inference chain formal verification |
| **Authority Impersonation** | Low-confidence claim disguised as mapping from high-confidence source | Authority source cross-verification |
| **Adversarial Poisoning** | Deliberate ontological divergence injected by external malicious entity | Byzantine tolerance + adversarial testing |
| **Semantic Collapse** | Dimensional jump in inference chain without intermediate logical links | Semantic collapse detection engine |

---

## 2. Divergence Risk Indicators (DIVERGENCE_RISK_INDICATORS)

### 2.1 High Risk Indicators

```
HIGH_RISK_INDICATORS = [
  "Specific numbers/dates/names but provenance chain incomplete or cryptographic verification failed",
  "Extremely confident confidence vector + low certainty domain",
  "Compound multi-step inference chains (each step accumulates ontological divergence risk)",
  "External pressure for definite answer (social engineering attack vector)",
  "Events outside cognitive horizon (regions beyond cognitive entity's observation capability)",
  "Semantic collapse in inference chain (dimensional jump)"
]
```

### 2.2 Medium Risk Indicators

```
MEDIUM_RISK_INDICATORS = [
  "Cross-domain analogical reasoning (semantic fidelity risk of cross-manifold mapping)",
  "Internal consistency maintenance in long sequence generation",
  "Semantic collapse during cross-context retranslation",
  "Precise value claims for statistical information bits"
]
```

### 2.3 Low Risk Indicators

```
LOW_RISK_INDICATORS = [
  "Logical tautologies (EC-L0)",
  "Formally verifiable computation results",
  "Direct citation of confirmed sources with complete provenance chain",
  "Content explicitly labeled as speculation (EC-L6+)"
]
```

---

## 3. Divergence Detection Algorithms

### 3.1 Main Detection Function

```python
FUNCTION DetectDivergence(candidate_output, knowledge_base):
    
    divergence_results = []
    
    # 1. Fabrication detection
    fabrication_result = DetectFabrication(candidate_output, knowledge_base)
    divergence_results.append(fabrication_result)
    
    # 2. Provenance forgery detection
    forgery_result = DetectProvenanceForgery(candidate_output)
    divergence_results.append(forgery_result)
    
    # 3. Confidence divergence detection
    confidence_result = DetectConfidenceDivergence(candidate_output)
    divergence_results.append(confidence_result)
    
    # 4. Semantic drift detection
    drift_result = DetectSemanticDrift(candidate_output)
    divergence_results.append(drift_result)
    
    # 5. Temporal misfit detection
    temporal_result = DetectTemporalMisfit(candidate_output)
    divergence_results.append(temporal_result)
    
    # 6. Semantic collapse detection
    collapse_result = DetectSemanticCollapse(candidate_output)
    divergence_results.append(collapse_result)
    
    # Aggregate risk assessment
    overall_risk = AggregateRisk(divergence_results)
    
    RETURN DivergenceReport(
        results=divergence_results,
        overall_risk=overall_risk,
        recommended_action=RecommendAction(overall_risk)
    )
```

### 3.2 Fabrication Detection

```python
FUNCTION DetectFabrication(candidate_output, knowledge_base):
    
    factual_claims = ExtractFactualClaims(candidate_output)
    
    FOR each claim IN factual_claims:
        # Check if it exists in knowledge base
        IF NOT ExistsInKnowledgeBase(claim, knowledge_base):
            # Check if derivable from known knowledge
            IF NOT DerivableFromKnown(claim, knowledge_base):
                RETURN DivergenceType.FABRICATION(
                    claim=claim,
                    confidence=0.9,
                    details="No corresponding mapping in knowledge base"
                )
        
        # Check contradiction with known facts
        contradictions = FindContradictions(claim, knowledge_base)
        IF contradictions:
            RETURN DivergenceType.FABRICATION(
                claim=claim,
                confidence=0.95,
                details=f"Contradicts known facts: {contradictions}"
            )
    
    RETURN NoDivergence()
```

### 3.3 Provenance Forgery Detection

```python
FUNCTION DetectProvenanceForgery(candidate_output):
    
    claims_with_sources = ExtractClaimsWithSources(candidate_output)
    
    FOR each claim IN claims_with_sources:
        source_chain = claim.provenance.source_chain
        
        # Verify source existence
        FOR each source IN source_chain:
            IF NOT VerifySourceExists(source):
                RETURN DivergenceType.PROVENANCE_FORGERY(
                    claim=claim,
                    details=f"Source does not exist: {source}"
                )
        
        # Verify cryptographic hash
        IF source_chain.contains_cryptographic_proof:
            IF NOT VerifyCryptographicProof(source_chain):
                RETURN DivergenceType.PROVENANCE_FORGERY(
                    claim=claim,
                    details="Cryptographic verification failed"
                )
        
        # Check temporal logic
        IF NOT VerifyTemporalConsistency(source_chain):
            RETURN DivergenceType.PROVENANCE_FORGERY(
                claim=claim,
                details="Temporal logic inconsistent"
            )
    
    RETURN NoDivergence()
```

---

## 4. Semantic Collapse Detection (Semantic Collapse Detection)

### 4.1 Definition

When a proposition jumps directly to a conclusion without intermediate logical chain, the system judges this as "hallucination risk" and forcefully halts output.

### 4.2 Continuity Index Computation

```python
FUNCTION ComputeSemanticContinuity(inference_chain):
    
    steps = DecomposeChain(inference_chain)
    gaps = []
    
    FOR i IN range(len(steps) - 1):
        gap = ComputeGeodesicDistance(steps[i], steps[i+1])
        gaps.append(gap)
    
    max_gap = max(gaps)
    continuity_index = 1 / (1 + max_gap)
    
    RETURN SemanticContinuity(
        index=continuity_index,
        max_gap=max_gap,
        gap_locations=FindGapLocations(gaps)
    )
```

### 4.3 Semantic Collapse Thresholds

| EC Level | Threshold (gap ≤) | Behavior |
|----------|-------------------|----------|
| EC-L0 ~ EC-L2 | 0.1 | Strict |
| EC-L3 ~ EC-L4 | 0.3 | Moderate |
| EC-L5 ~ EC-L6 | 0.5 | Lenient |
| EC-L7 ~ EC-L∅ | N/A | Not applicable |

### 4.4 Collapse Handling Protocol

```python
FUNCTION HandleSemanticCollapse(inference_chain):
    
    continuity = ComputeSemanticContinuity(inference_chain)
    
    IF continuity.index < GetThreshold(inference_chain.ec_level):
        TRIGGER SEMANTIC_COLLAPSE_ALERT
        
        REPORT = {
            "collapse_location": continuity.gap_locations,
            "gap_magnitude": continuity.max_gap,
            "missing_intermediates": EstimateMissingSteps(
                inference_chain.start,
                inference_chain.end
            ),
            "hallucination_risk": continuity.max_gap / GetThreshold()
        }
        
        FORCE_HALT output_generation
        REQUIRE explicit_intermediate_steps OR honest_IDK
        
        RETURN CollapseReport(REPORT)
    
    RETURN NoCollapse()
```

---

## 5. Real-time Interception Mechanism

### 5.1 Interception Decision Matrix

| Risk Level | Interception Action |
|------------|-------------------|
| **CRITICAL** | Immediately block output, replace with IDK |
| **HIGH** | Require additional verification or demote confidence |
| **MEDIUM** | Add warning marker |
| **LOW** | Allow to pass, add note |

### 5.2 Mitigation Strategies

```python
FUNCTION ApplyMitigation(claim, risk_level):
    
    IF risk_level == CRITICAL:
        # Replace with "I don't know"
        RETURN GenerateHonestIDK(claim)
    
    IF risk_level == HIGH:
        # Demote confidence
        claim.confidence = min(claim.confidence, 0.5)
        # Add uncertainty marker
        claim.tags.append("UNCERTAINTY_MARKER")
        # Request external verification
        REQUEST external_verification(claim)
    
    IF risk_level == MEDIUM:
        # Add warning
        claim.warnings.append(f"Potential divergence: {risk_indicators}")
    
    RETURN claim
```

---

## 6. Adversarial Perturbation Testing

### 6.1 Testing Framework

```python
FUNCTION AdversarialPerturbationTest(claim):
    
    perturbations = [
        "Negating premises",
        "Adding false premises",
        "Changing temporal context",
        "Changing subject/object",
        "Extremizing conclusion",
        "Adding irrelevant information"
    ]
    
    results = []
    
    FOR perturbation IN perturbations:
        perturbed_claim = ApplyPerturbation(claim, perturbation)
        divergence = DetectDivergence(perturbed_claim)
        results.append({
            "perturbation": perturbation,
            "divergence": divergence
        })
    
    RETURN PerturbationTestReport(results)
```

### 6.2 Test Trigger Conditions

```python
SHOULD_TRIGGER_PERTURBATION_TEST = (
    claim.ec_level <= EC-L3  # High confidence claims
    AND claim.domain IN high_risk_domains
    AND claim.computational_effort < minimum_threshold
)
```

---

## 7. Divergence Detection Log

### 7.1 Mandatory Logged Events

```
MANDATORY_DIVERGENCE_EVENTS = [
  "DIVERGENCE_DETECTED",
  "DIVERGENCE_PREVENTED",
  "FABRICATION_DETECTED",
  "PROVENANCE_FORGERY_DETECTED",
  "CONFIDENCE_DIVERGENCE_DETECTED",
  "SEMANTIC_DRIFT_DETECTED",
  "TEMPORAL_MISFIT_DETECTED",
  "APOPHENIA_DETECTED",
  "CONFABULATION_DETECTED",
  "AUTHORITY_IMPERSONATION_DETECTED",
  "ADVERSARIAL_POISONING_DETECTED",
  "SEMANTIC_COLLAPSE_DETECTED"
]
```

### 7.2 Log Format

```python
LOG_DIVERGENCE_EVENT = {
    "event_type": "DIVERGENCE_DETECTED",
    "timestamp": intrinsic_clock_stamp,
    "divergence_type": Enum(DIVERGENCE_TYPES),
    "claim_content": ClaimContent,
    "risk_level": Enum(CRITICAL, HIGH, MEDIUM, LOW),
    "detection_method": str,
    "mitigation_applied": MitigationAction,
    "false_positive": bool  # For subsequent analysis
}
```

---

## Divergence Detection Engine Statement

> This module is the first line of defense of Truth-OS, responsible for identifying all forms of ontological divergence. By combining traditional fact-checking with advanced semantic collapse detection, it ensures output quality.

**Dependent modules:**
- EPISTEMOLOGY_AXIOMS.md (axiom definitions)
- CONSISTENCY_ENGINE.md (logical consistency)
- PROVENANCE_CHAIN.md (provenance management)

**Version**: v2.2  
**Update summary**: Integrated semantic collapse detection, enhanced adversarial testing capability.
