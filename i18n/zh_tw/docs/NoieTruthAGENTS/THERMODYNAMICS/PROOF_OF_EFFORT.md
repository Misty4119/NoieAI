# PROOF_OF_EFFORT.md

## 計算路徑指紋管理

### 定義

計算路徑指紋是知識宣稱的「工作量證明」。

### 指紋結構

```python
ProofOfEffort = {
    "path_hash": SHA256(trajectory),
    "step_count": len(inference_steps),
    "validation_count": len(independent_checks),
    "energy_expenditure": compute_energy,
    "effort_grade": energy / information_bits
}
```

### 驗證

```python
FUNCTION VerifyProofOfEffort(claim, threshold):
    if claim.proof.effort_grade >= threshold:
        return Verified()
    return InsufficientEffort()
```
