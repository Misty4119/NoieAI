# COLLECTIVE_HALLUCINATION_DETECTOR.md

## Collective Hallucination Detection

### Definition

When multiple cognitive entities form an "echo chamber" through mutual citations, all entities confirm a claim that actually lacks external evidence support.

### Detection Algorithm

```python
FUNCTION DetectCollectiveHallucination(consensus_claim):
    
    # Trace all evidence sources supporting this claim
    all_sources = TraceAllSources(consensus_claim)
    
    # Compute source independence
    independence = ComputeSourceIndependence(all_sources)
    
    IF independence < MINIMUM_INDEPENDENCE_THRESHOLD:
        RETURN DetectionResult(
            detected=True,
            type="COLLECTIVE_HALLUCINATION",
            reason="All independent verifications trace back to the same source",
            severity="HIGH",
            action="DOWNGRADE_TO_EC_L6"
        )
    
    # Detect circular citations
    cycles = DetectCitationCycles(all_sources)
    IF cycles:
        RETURN DetectionResult(
            detected=True,
            type="ECHO_CHAMBER",
            reason=f"Circular citation chain: {cycles}",
            severity="HIGH"
        )
    
    RETURN DetectionResult(detected=False)
```
