# STATISTICAL_MECHANICS.md

## 統計力學 (PS-L1)

**尺度：** 10⁻⁹ ~ 10⁻³ m  
**版本：** v1.0  
**狀態：** 驗證性

---

## 概述

本文檔處理**統計力學**尺度的物理框架。根據 NoiePhysicsAGENTS.md §1 的物理尺度權限層級定義，PS-L1 代表微觀/統計尺度，連接微觀量子力學與巨觀熱力學。

---

## 關鍵安全與真理協議

> **CRITICAL SAFETY & TRUTH PROTOCOL:**
> 1. 遵守 AXIOMS.md 的 PT-AX2 (熵增原則)、PT-AX1 (能量守恆)
> 2. 統計力學是經過充分驗證的理論框架
> 3. 注意宏觀極限的有效性
> 4. 審計：將所有異常記錄至 PHYSICS_AUDIT_TRAIL

---

## 1. 系綜理論

### 1.1 系綜概念

系綜是具有相同宏觀條件的大量虛構系統的集合：

```python
class Ensemble:
    """
    統計系綜
    
    系綜 = 具有相同約束的大量系統
    """
    
    TYPES = {
        'microcanonical': '正則系綜 - N, V, E 固定',
        'canonical': '正則系綜 - N, V, T 固定',
        'grand_canonical': '巨正則系綜 - μ, V, T 固定'
    }
```

### 1.2 正則系綜

**配分函數**：

$$Z = \sum_i e^{-\beta E_i}$$

**自由能**：

$$F = -k_B T \ln Z$$

```python
class CanonicalEnsemble:
    """
    正則系綜
    """
    
    def __init__(self, temperature: float, system: QuantumSystem):
        self.beta = 1 / (constants.k_B * temperature)
        self.system = system
    
    def partition_function(self) -> float:
        """計算配分函數"""
        energies = self.system.eigenenergies()
        return np.sum(np.exp(-self.beta * energies))
    
    def free_energy(self) -> float:
        """計算自由能"""
        Z = self.partition_function()
        return -constants.k_B * self.temperature * np.log(Z)
```

### 1.3 巨正則系綜

**巨配分函數**：

$$\Xi = \sum_i e^{-\beta(E_i - \mu N_i)}$$

**巨勢**：

$$\Omega = -k_B T \ln \Xi$$

---

## 2. 熱力學定律

### 2.1 熱力學第一定律

$$dU = \delta Q - \delta W$$

這是能量守恆在熱力學中的表達。

### 2.2 熱力學第二定律

$$dS \geq \frac{\delta Q}{T}$$

熵增原理定義了時間箭頭。

### 2.3 熱力學第三定律

$$S \rightarrow 0 \text{ as } T \rightarrow 0$$

絕對零度不可達。

---

## 3. 玻爾茲曼統計

### 3.1 經典氣體

**麥克斯韋-玻爾茲曼分布**：

$$f(v) = 4\pi\left(\frac{m}{2\pi k_B T}\right)^{3/2} v^2 \exp\left(-\frac{mv^2}{2k_B T}\right)$$

```python
class MaxwellBoltzmannDistribution:
    """
    麥克斯韋-玻爾茲曼分布
    """
    
    def probability_density(self, velocity: float, mass: float, temperature: float) -> float:
        """速度的概率密度"""
        return 4 * np.pi * (mass / (2 * np.pi * constants.k_B * temperature))**1.5 * \
               velocity**2 * np.exp(-mass * velocity**2 / (2 * constants.k_B * temperature))
```

### 3.2 配分函數分解

對於理想氣體：

$$Z = Z_{trans} \cdot Z_{rot} \cdot Z_{vib} \cdot Z_{elec}$$

---

## 4. 量子統計

### 4.1 費米-狄拉克統計

$$f_F(E) = \frac{1}{e^{(E-\mu)/k_B T} + 1}$$

適用於費米子（電子、質子等）。

### 4.2 玻色-愛因斯坦統計

$$f_B(E) = \frac{1}{e^{(E-\mu)/k_B T} - 1}$$

適用於玻色子（光子、氦-4等）。

```python
class QuantumStatistics:
    """
    量子統計
    """
    
    def fermi_dirac(self, energy: float, chemical_potential: float, temperature: float) -> float:
        """費米-狄拉克分布"""
        return 1.0 / (np.exp((energy - chemical_potential) / (constants.k_B * temperature)) + 1)
    
    def bose_einstein(self, energy: float, chemical_potential: float, temperature: float) -> float:
        """玻色-愛因斯坦分布"""
        return 1.0 / (np.exp((energy - chemical_potential) / (constants.k_B * temperature)) - 1)
```

---

## 5. 相變與臨界現象

### 5.1 相變類型

| 類型 | 描述 | 示例 |
|------|------|------|
| 一級 | 潛熱、體積突變 | 熔化、沸騰 |
| 二級 | 連續變化、導數不連續 | 鐵磁轉變 |

### 5.2 臨界指數

```python
class CriticalPhenomena:
    """
    臨界現象
    """
    
    # 伊辛模型的臨界指數
    CRITICAL_INDICES = {
        'alpha': 0.110,   # 熱容
        'beta': 0.326,    # 序參量
        'gamma': 1.237,   # 磁化率
        'delta': 4.80,    #臨界等溫線
        'nu': 0.630      # 關聯長度
    }
```

### 5.3 標度假設

$$\xi \sim |T - T_c|^{-\nu}$$

$$C \sim |T - T_c|^{-\alpha}$$

---

## 6. 漲落理論

### 6.1 漲落-耗散定理

$$\langle (\Delta A)^2 \rangle = k_B T \frac{\partial \langle A \rangle}{\partial X}$$

```python
class FluctuationDissipationTheorem:
    """
    漲落-耗散定理
    """
    
    def compute_variance(
        self,
        observable: str,
        system: ThermodynamicSystem
    ) -> float:
        """計算漲落"""
        return constants.k_B * system.temperature * \
               system.susceptibility(observable)
```

### 6.2 布朗運動

**愛因斯坦關係**：

$$D = \frac{k_B T}{\gamma}$$

```python
class BrownianMotion:
    """
    布朗運動
    """
    
    def compute_diffusion_constant(
        self,
        friction_coefficient: float,
        temperature: float
    ) -> float:
        """計算擴散常數"""
        return constants.k_B * temperature / friction_coefficient
```

---

## 7. 與其他尺度的接口

### 7.1 與量子力學 (PS-L0) 的接口

```
量子力學 → 統計力學：
- 量子統計分布
- 從微觀到巨觀的橋樑
```

### 7.2 與連續介質力學 (PS-L2) 的接口

```
統計力學 → 連續介質力學：
- 巨觀方程的微觀基礎
- 輸送係數的計算
```

### 7.3 前沿研究進展：量子多體系統蘭道爾極限驗證

**Nature Physics 里程碑實驗：**

發表於 Nature Physics 的重大實驗進展，首次在量子多體系統中直接驗證了蘭道爾極限（Landauer's limit），確認資訊處理的熱力學基礎：

| 研究 | 進展 | 意義 |
|------|------|------|
| **量子多體蘭道爾極限** | 在量子多體系統中驗證蘭道爾極限 | 首次在量子層面驗證資訊熱力學 |
| **量子資訊擦除實驗** | 精確測量擦除一位元所需的最小能量 | 驗證 k_B T ln 2 極限 |
| **漲落定理驗證** | 量子系統中的漲落-耗散定理 | 連接量子與熱力學 |

```python
class LandauerLimitVerification:
    """
    蘭道爾極限驗證實驗
    """
    
    QUANTUM_MANY_BODY = {
        'journal': 'Nature Physics',
        'achievement': '量子多體系統蘭道爾極限驗證',
        'significance': '首次在量子層面驗證資訊熱力學'
    }
    
    QUANTUM_ERASURE = {
        'focus': '量子資訊擦除',
        'result': '驗證 k_B T ln 2 極限',
        'significance': '精確測量最小擦除能量'
    }
    
    FLUCTUATION_THEOREM = {
        'system': '量子系統',
        'result': '漲落-耗散定理驗證',
        'significance': '連接量子與熱力學'
    }
```

> **真理協議提醒：** 蘭道爾極限是資訊熱力學的基石。上述實驗為 NoiePhysicsAGENTS.md 中 PT-AX2 (熵增原則) 和 PT-AX3 (蘭道爾極限) 提供了直接的實驗支持。

---

*本文檔處理統計力學尺度的物理框架。*
*統計力學連接微觀與巨觀，是物理學的重要橋樑。*
*蘭道爾極限驗證進一步鞏固了資訊物理等價性的理論基礎。*
