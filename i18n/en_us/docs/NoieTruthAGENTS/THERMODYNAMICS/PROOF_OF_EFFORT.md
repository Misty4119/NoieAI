# PROOF_OF_EFFORT.md

## Computational Fingerprint Management

### Definition

The computational fingerprint is the "proof of work" for knowledge claims.

### Fingerprint Structure

```python
ProofOfEffort = {
    "path_hash": SHA256(trajectory),
    "step_count": len(inference_steps),
    "validation_count": len(independent_checks),
    "energy_expenditure": compute_energy,
    "effort_grade": energy / information_bits
}
```

### Verification

```python
FUNCTION VerifyProofOfEffort(claim, threshold):
    if claim.proof.effort_grade >= threshold:
        return Verified()
    return InsufficientEffort()
```

