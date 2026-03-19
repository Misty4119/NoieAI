# QUANTUM_MECHANICS.md

## 量子力學 (PS-L0)

**尺度：** 10⁻³⁵ ~ 10⁻⁹ m  
**版本：** v1.0  
**狀態：** 驗證性

---

## 概述

本文檔處理**量子力學**尺度的物理框架。根據 NoiePhysicsAGENTS.md §1 的物理尺度權限層級定義，PS-L0 代表量子尺度，涵蓋從原子到分子的物理現象。

量子力學是現代物理學最成功和經過最充分驗證的理論之一。

---

## 關鍵安全與真理協議

> **CRITICAL SAFETY & TRUTH PROTOCOL:**
> 1. 遵守 AXIOMS.md 的 PT-AX21 (測量反作用)、PT-AX22 (不確定性原理)
> 2. 事實區分：量子力學是經過驗證的理論，但需注意適用範圍
> 3. 承認測量問題仍未完全解決
> 4. 審計：將所有異常記錄至 PHYSICS_AUDIT_TRAIL

---

## 1. 基本原理

### 1.1 波函數與態向量

量子力學的基本假設是系統的狀態用希爾伯特空間中的向量描述：

```python
class QuantumState:
    """
    量子態
    
    用向量 |ψ⟩ 或密度矩陣 ρ 描述。
    """
    
    def __init__(self, ket: StateVector):
        self.ket = ket  # 態向量
        self.dimension = ket.shape[0]
    
    def probability(self, observable: Operator, eigenvalue: float) -> float:
        """波恩規則：P = |⟨ψ|O|ψ⟩|²"""
        projector = self._get_projector(observable, eigenvalue)
        return abs(self.ket.conj().T @ projector @ self.ket)**2
```

### 1.2 薛丁格方程

**含時薛丁格方程**：

$$i\hbar\frac{\partial}{\partial t}|\psi(t)\rangle = \hat{H}|\psi(t)\rangle$$

**不含時薛丁格方程**：

$$\hat{H}|\psi_n\rangle = E_n|\psi_n\rangle$$

```python
class SchrodingerEquation:
    """
    薛丁格方程求解器
    """
    
    def solve_time_dependent(
        self,
        hamiltonian: Operator,
        initial_state: QuantumState,
        time_span: Tuple[float, float]
    ) -> List[QuantumState]:
        """
        求解含時薛丁格方程
        
        方法：Runge-Kutta, Crank-Nicolson, split-operator
        """
        pass
    
    def solve_time_independent(
        self,
        hamiltonian: Operator
    ) -> List[EigenState]:
        """
        求解不含時薛丁格方程
        
        方法：精確對角化、變分法、擾動論
        """
        pass
```

---

## 2. 不確定性原理

### 2.1 海森堡不確定性原理

$$\Delta x \cdot \Delta p \geq \frac{\hbar}{2}$$

```python
class UncertaintyRelation:
    """
    不確定性關係
    """
    
    def compute_position_momentum(
        self,
        state: QuantumState
    ) -> float:
        """計算位置-動量不確定性乘積"""
        x_var = state.expectation_value(Operator('x**2')) - \
                state.expectation_value(Operator('x'))**2
        p_var = state.expectation_value(Operator('p**2')) - \
                state.expectation_value(Operator('p'))**2
        return np.sqrt(x_var * p_var)
```

### 2.2 能量-時間不確定性

$$\Delta E \cdot \Delta t \geq \frac{\hbar}{2}$$

這不是算符關係，而是針對特定過程的統計表述。

---

## 3. 量子測量

### 3.1 測量假設

測量對量子態有不可逆的影響：

```python
class QuantumMeasurement:
    """
    量子測量
    """
    
    def projective_measure(
        self,
        state: QuantumState,
        observable: Operator
    ) -> MeasurementOutcome:
        """
        投影測量
        
        1. 計算本徵值和本徵態
        2. 根據波恩規則選擇結果
        3. 狀態塌縮到對應本徵態
        """
        eigenvalues, eigenvectors = np.linalg.eigh(observable.matrix)
        probabilities = abs(eigenvectors.conj().T @ state.ket)**2
        outcome_idx = np.random.choice(len(probabilities), p=probabilities)
        
        return MeasurementOutcome(
            eigenvalue=eigenvalues[outcome_idx],
            post_state=QuantumState(eigenvectors[:, outcome_idx]),
            probability=probabilities[outcome_idx]
        )
    
    def weak_measure(
        self,
        state: QuantumState,
        observable: Operator,
        strength: float
    ) -> WeakMeasurementOutcome:
        """
        弱測量
        
        減少干擾但增加噪聲
        """
        pass
```

### 3.2 測量問題

量子測量的物理機制仍是未解問題：

| 詮釋 | 測量描述 |
|------|----------|
| 哥本哈根 | 波函數在測量時「塌縮」 |
| 多世界 | 所有結果都發生在分支宇宙中 |
| 導航波 | 粒子有確定軌跡，波函數引導 |
| 客觀塌縮 | 物理過程導致自發塌縮 |

---

## 4. 量子態與糾纏

### 4.1 量子態類型

| 態類型 | 描述 | 示例 |
|--------|------|------|
| 純態 | 可以用單一態向量描述 | \|ψ⟩ = α\|0⟩ + β\|1⟩ |
| 混合態 | 需要密度矩陣 | ρ = Σ pᵢ\|ψᵢ⟩⟨ψᵢ\| |
| 糾纏態 | 不可分離的態 | \|Φ⁺⟩ = (|00⟩ + |11⟩)/√2 |

### 4. 糾纏與貝爾不等式

```python
class Entanglement:
    """
    量子糾纏
    """
    
    def compute_concurrence(self, state: DensityMatrix) -> float:
        """計算糾纏 concurrence"""
        # 適用於 2- qubit 系統
        pass
    
    def verify_bell_inequality(
        self,
        correlations: dict
    ) -> BellTestResult:
        """
        驗證貝爾不等式
        
        貝爾不等式被違反 → 不存在局部隱藏變量
        """
        S = correlations['S']
        # CHSH 不等式：|S| ≤ 2
        # 量子力學預測：|S| ≤ 2√2 ≈ 2.828
        return BellTestResult(
            violated=abs(S) > 2,
            value=S,
            classical_bound=2,
            quantum_bound=2*np.sqrt(2)
        )
```

---

## 5. 量子動力學

### 5.1 態演化

**么正演化**：

$$|\psi(t)\rangle = U(t, t_0)|\psi(t_0)\rangle$$

其中 $U(t, t_0) = \exp\left(-\frac{i}{\hbar}\int_{t_0}^t H(t')dt'\right)$

```python
class UnitaryEvolution:
    """
    么正演化
    """
    
    def compute_propagator(
        self,
        hamiltonian: Operator,
        time_step: float
    ) -> Propagator:
        """
        計算傳播子
        
        方法：
        - 直接指數：exp(-iHt/ħ)
        - Trotter 分解
        - Magnus 展開
        """
        return Propagator(time_step)
```

### 5.2 開放量子系統

當系統與環境交互時，需要考慮退相干：

```python
class OpenQuantumSystem:
    """
    開放量子系統
    """
    
    def apply_decoherence(
        self,
        state: DensityMatrix,
        environment: Environment,
        time: float
    ) -> DensityMatrix:
        """
        應用退相干
        
        計算環境導致的糾纏喪失
        """
        pass
    
    def compute_decoherence_time(
        self,
        system: QuantumSystem,
        environment: Environment
    ) -> float:
        """
        計算退相干時間尺度
        """
        pass
```

---

## 6. 量子應用

### 6.1 量子計算

| 量子門 | 矩陣 | 描述 |
|--------|------|------|
| Pauli-X | [[0,1],[1,0]] | 量子 NOT |
| Pauli-Y | [[0,-i],[i,0]] | 相位翻轉 |
| Pauli-Z | [[1,0],[0,-1]] | 相位翻轉 |
| Hadamard | [[1,1],[1,-1]]/√2 | 疊加 creation |
| CNOT | 控制-NOT 門 | 2-qubit 門 |

### 6.2 量子模擬

冷原子、超導量子比特、離子阱等平台可以用於模擬其他量子系統。

### 6.3 前沿研究進展

**量子硬體突破：**

| 機構 | 進展 | 意義 |
|------|------|------|
| **Quantinuum** | 94個受保護的邏輯量子位元 | 達到實用級錯誤修正規模 |
| **Quantum Elements** | 91-94% fidelity | 高保真量子操作的新標準 |
| **IBM** | 127量子位元處理器 | 超導量子計算的规模化里程碑 |
| **Google** | AlphaQubit 2 (AI解碼器) | 機器學習輔助的量子錯誤解碼 |

```python
class QuantumComputingAdvances:
    """
    量子計算最新進展
    """
    
    QUANTINUUM = {
        'logical_qubits': 94,
        'technology': 'trapped_ion',
        'significance': '實用級錯誤修正規模'
    }
    
    QUANTUM_ELEMENTS = {
        'fidelity_range': (0.91, 0.94),
        'technology': 'quantum_dot',
        'significance': '高保真量子操作'
    }
    
    IBM = {
        'qubits': 127,
        'technology': 'superconducting',
        'significance': '超導量子計算規模化'
    }
    
    GOOGLE_ALPHAQUBIT_2 = {
        'type': 'AI_decoder',
        'significance': 'ML輔助量子錯誤解碼'
    }
```

> **真理協議提醒：** 量子硬體發展迅速，以上數據基於發布時的官方聲明。實際系統效能可能因環境條件而異。

---

## 7. 與其他尺度的接口

### 7.1 與統計力學 (PS-L1) 的接口

```
量子力學 → 統計力學：
- 量子統計分布 (Fermi-Dirac, Bose-Einstein)
- 從量子態推導巨觀性質
```

### 7.2 與量子重力 (PS-L(-1)) 的接口

```
量子力學 → 量子重力：
- 需要與廣義相對論統一
- 普朗克尺度需要量子引力
```

---

*本文檔處理量子力學尺度的物理框架。*
*量子力學是經過充分驗證的理論，適用於 PS-L0 尺度的物理現象。*
