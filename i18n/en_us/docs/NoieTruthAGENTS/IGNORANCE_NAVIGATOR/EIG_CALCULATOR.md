# EIG_CALCULATOR.md

## Expected Information Gain Calculation

### Definition

Expected Information Gain (EIG) measures the amount of information an observation can bring.

$$EIG(x, o) = H[p(x)] - E_{p(y|o)}[H[p(x|y)]]$$

### Implementation

```python
FUNCTION CalculateEIG(observation, current_belief):
    # Compute entropy before observation
    H_before = Entropy(current_belief)
    
    # Compute expected entropy after observation
    possible_outcomes = observation.get_possible_outcomes()
    H_after = 0
    
    for outcome in possible_outcomes:
        posterior = UpdateBelief(current_belief, outcome)
        prob = outcome.probability
        H_after += prob * Entropy(posterior)
    
    # Compute information gain
    EIG = H_before - H_after
    
    return EIG
```

### Application

Used to select the most valuable direction for the next exploration step.
