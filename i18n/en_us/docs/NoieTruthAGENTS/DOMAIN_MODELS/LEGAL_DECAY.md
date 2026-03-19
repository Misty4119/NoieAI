# LEGAL_DECAY.md

## Legal Domain Knowledge Decay Law

### Domain Characteristics

| Feature | Description |
|---------|-------------|
| **Regulatory Stability** | Relatively stable but with amendment cycles |
| **Precedent Accumulation Effect** | Old precedents may be overturned by new ones |
| **Temporal Validity** | Laws have statutes of limitations |
| **Territoriality** | Different jurisdictions have different regulations |

### Decay Constants

$$\lambda^*_{\text{legal}} \approx 0.05 - 0.15$$

### Legal Sub-Domain Decay Characteristics

| Sub-Domain | λ* Range | Decay Triggers |
|-----------|-----------|----------------|
| Constitutional Law | 0.01 - 0.05 | Constitutional amendments |
| Criminal Law | 0.02 - 0.08 | Criminal penalty amendments |
| Civil Law | 0.03 - 0.10 | Civil law amendments |
| Administrative Law | 0.05 - 0.15 | Regulatory amendments |
| International Law | 0.02 - 0.08 | Treaty revisions |

### Decay Trigger Conditions

```python
LEGAL_DECAY_TRIGGERS = [
    "Passage of legal amendments",
    "Changes in judicial interpretation",
    "Precedent reversal",
    "Statute of limitations expiration",
    "Jurisdiction changes",
    "Treaty withdrawal/accession"
]
```

### Calculation Formula

```python
FUNCTION ComputeLegalValidity(claim, current_date, jurisdiction):
    time_delta = ComputeTimeSinceClaim(current_date, claim.effective_date)
    law_changes = CountLawChanges(claim.law_reference, jurisdiction)
    decay = exp(-lambda_legal * (time_delta + law_changes))
    RETURN min(decay * claim.original_reliability, 1.0)
```
