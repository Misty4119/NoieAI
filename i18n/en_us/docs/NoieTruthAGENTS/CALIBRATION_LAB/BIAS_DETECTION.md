# BIAS_DETECTION.md

## Systematic Bias Detection

### Bias Type Definitions

| Bias Type | Symbolic Description | Typical Manifestation |
|-----------|---------------------|-----------------------|
| **Overconfidence** | C > A systematically | Overestimating own abilities |
| **Underconfidence** | C < A systematically | Underestimating own abilities |
| **Domain Bias** | Specific domains C ≠ A | Systematic inaccuracy in certain domains |
| **Availability Bias** | Tendency to remember easily accessible information | Ignoring rare but important information |
| **Confirmation Bias** | Tendency to support existing beliefs | Ignoring counterexamples |
| **Anchoring Bias** | Over-reliance on first information | Subsequent judgments influenced by initial value |
| **Hindsight Bias** | Believing past events were predictable | Underestimating randomness |

### Detection Algorithms

```python
FUNCTION DetectBiases(claim_history):
    
    biases_detected = {}
    
    # 1. Overconfidence detection
    overconfidence = DetectOverconfidence(claim_history)
    biases_detected["overconfidence"] = overconfidence
    
    # 2. Underconfidence detection
    underconfidence = DetectUnderconfidence(claim_history)
    biases_detected["underconfidence"] = underconfidence
    
    # 3. Domain bias detection
    domain_biases = DetectDomainBiases(claim_history)
    biases_detected["domain_biases"] = domain_biases
    
    # 4. Availability bias detection
    availability_bias = DetectAvailabilityBias(claim_history)
    biases_detected["availability"] = availability_bias
    
    # 5. Confirmation bias detection
    confirmation_bias = DetectConfirmationBias(claim_history)
    biases_detected["confirmation"] = confirmation_bias
    
    RETURN BiasReport(biases_detected)
```

### Bias Correction

```python
FUNCTION CorrectBiases(bias_report):
    
    corrections = {}
    
    IF bias_report.overconfidence.detected:
        corrections["confidence_adjustment"] = -bias_report.overconfidence.magnitude
        NOTIFY "Systematic overconfidence detected, confidence adjusted"
    
    IF bias_report.underconfidence.detected:
        corrections["confidence_adjustment"] = +bias_report.underconfidence.magnitude
        NOTIFY "Systematic underconfidence detected, confidence adjusted"
    
    FOR domain IN bias_report.domain_biases:
        corrections[domain] = CalculateDomainCorrection(domain)
    
    RETURN corrections
```
