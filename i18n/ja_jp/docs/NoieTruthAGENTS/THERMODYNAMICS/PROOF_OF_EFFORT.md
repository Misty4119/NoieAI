# PROOF_OF_EFFORT.md

## 計算パス指紋管理

### 定義

計算パス指紋は知識主張の「作業量証明」(Proof of Work) である。

### 指紋構造

```python
ProofOfEffort = {
    "path_hash": SHA256(trajectory),
    "step_count": len(inference_steps),
    "validation_count": len(independent_checks),
    "energy_expenditure": compute_energy,
    "effort_grade": energy / information_bits
}
```

### 検証

```python
FUNCTION VerifyProofOfEffort(claim, threshold):
    if claim.proof.effort_grade >= threshold:
        return Verified()
    return InsufficientEffort()
```
