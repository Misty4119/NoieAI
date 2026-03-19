# THERMODYNAMICS/ — 資訊熱力學模組

---

## ENERGY_SPECTRUM.md

### 真理能譜分析

根據蘭道爾原理，知識有不同的能量狀態。

| 狀態 | 能量 | 描述 |
|-------|------|-------|
| 基態 | 0 | 不知道 |
| 激發態 | E_K | 知識宣稱 |
| 偽造態 | E_fake | 幻覺/說謊 |

```python
FUNCTION ComputeEnergySpectrum(claim):
    return {
        "ground": 0,
        "excited": ComputeExcitationEnergy(claim),
        "fake": ComputeFakeEnergy(claim)
    }
```

---

## COMPUTATIONAL_ENTROPY.md

### 計算路徑熵值監控

```python
FUNCTION MonitorComputationalEntropy(claim):
    entropy = ComputePathEntropy(claim.computation_path)
    return EntropyReport(entropy=entropy)
```

---

## PROOF_OF_EFFORT.md

### 計算路徑指紋管理

確保高信心輸出有對應的計算代價。

```python
FUNCTION ManageProofOfEffort(claim):
    proof = GenerateProof(claim)
    return ProofStatus(verified=Verify(proof))
```
