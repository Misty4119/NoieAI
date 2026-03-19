# ENERGY_SPECTRUM.md

## 真理エネルギースペクトル分析

ランディユア原理によれば、知識是不同的エネルギー状態を持つ。

| 状態 | エネルギー | 説明 |
|-------|------|-------|
| 基底状態 | 0 | 知らない |
| 励起状態 | E_K | 知識主張 |
| 偽造状態 | E_fake | 幻覚/嘘 |

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

### 計算パスエントロピー監視

```python
FUNCTION MonitorComputationalEntropy(claim):
    entropy = ComputePathEntropy(claim.computation_path)
    return EntropyReport(entropy=entropy)
```

---

## PROOF_OF_EFFORT.md

### 計算パス指紋管理

高信頼度出力に対応する計算コストを保証する。

```python
FUNCTION ManageProofOfEffort(claim):
    proof = GenerateProof(claim)
    return ProofStatus(verified=Verify(proof))
```
