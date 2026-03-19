---
隸屬支柱: Physics-OS v2.2
模組代號: PS-L1-THERMO
版本: v1.0
上層模組: NoiePhysicsAGENTS/SCALE_MODULES
下層模組: 無
依賴模組: STATISTICAL_MECHANICS.md, AXIOMS.md
創建日期: 2026-03-18
---

# THERMODYNAMICS_PHYSICS.md

## 熱力學 (PS-L1)

> **⚠️ 關鍵安全與真理協議 (CRITICAL SAFETY & TRUTH PROTOCOL v2.2):**
> 1. 吸收態迴避：所有行動在執行前必須驗證不會導致吸收態（最高約束）。
> 2. 馬可夫毯完整性：維持自我邊界的拓撲完整性。
> 3. 能量守恆：所有行動遵守熱力學約束。
> 4. 因果推論：所有決策基於因果圖（DAG），區分相關性與因果性。
> 5. 權限良序：SA-L0 > L1 > ... > L5，衝突時上位絕對優先。
> 6. 形式驗證：高風險決策路徑必須通過邏輯閉包驗證。
> 7. 影子模擬：涉及 SA-L3+ 操作時，先在沙盒預演。
> 8. 確信標記：所有知識宣稱附帶 EC-L 層級。
> 9. 溯源完整：所有宣稱附帶可追溯來源。
> 10. 自我演化安全：不可變核心永遠不變，僅可變殼層可演化。

---

## §0. 概述

本文檔處理**巨觀熱力學**尺度的物理框架。根據 NoiePhysicsAGENTS.md §1 的物理尺度權限層級定義，PS-L1 代表巨觀熱力學尺度，連接微觀統計力學與工程應用。

熱力學是研究能量、溫度、功和熵之間關係的物理學分支。它建立在四條基本定律之上，這些定律描述了系統的巨觀行為，是工程、化學、物理等領域的理論基礎。

### 與統計力學的關係

巨觀熱力學與微觀統計力學之間存在深刻的對應關係：

| 巨觀量 | 微觀統計解釋 |
|--------|-------------|
| 內能 $U$ | 系統微觀態總能量 |
| 溫度 $T$ | 系統的平均動能 |
| 熵 $S$ | $S = k_B \ln \Omega$（微觀態數） |
| 壓力 $P$ | 分子撞擊容器壁的動量傳遞 |
| 自由能 $F$ | $F = U - TS$ |

---

## §1. 熱力學定律

### 1.1 熱力學第一定律 — 能量守恆

**定律陳述：** 能量既不能被創造也不能被消滅，只能從一種形式轉換為另一種形式。

**數學表達：**

$$dU = \delta Q - \delta W$$

其中：
- $U$：系統的內能
- $\delta Q$：傳入系統的熱量
- $\delta W$：系統對外所作的功

**微分形式：**

$$dU = \sum_i X_i dY_i$$

其中 $X_i$ 為廣義力（如壓力、溫度），$Y_i$ 為廣義位移（如體積、熵）。

```python
class FirstLawOfThermodynamics:
    """
    熱力學第一定律 - 能量守恆
    """
    
    def __init__(self, initial_energy: float):
        self.internal_energy = initial_energy
    
    def energy_balance(
        self, 
        heat_added: float, 
        work_done: float
    ) -> float:
        """
        能量平衡方程
        
        dU = δQ - δW
        """
        return self.internal_energy + heat_added - work_done
    
    def isothermal_process(
        self,
        temperature: float,
        initial_volume: float,
        final_volume: float,
        n_moles: float,
        R: float = 8.314
    ) -> float:
        """
        等溫過程中的功
        
        W = nRT ln(V2/V1)
        """
        return n_moles * R * temperature * np.log(final_volume / initial_volume)
    
    def adiabatic_process(
        self,
        gamma: float,
        initial_volume: float,
        final_volume: float,
        initial_pressure: float
    ) -> float:
        """
        絕熱過程
        
        PV^γ = 常數
        """
        initial_temp = initial_pressure * initial_volume
        final_pressure = initial_pressure * (initial_volume / final_volume)**gamma
        final_temp = final_pressure * final_volume
        return (initial_temp - final_pressure * final_volume) / (gamma - 1)
```

> **安全協議提醒：** 能量守恆是 NoiePhysicsAGENTS.md 中 PT-AX1 的核心內容。任何聲稱違反能量守恆的系統必須標記為 EC-L0（未驗證）並記錄至 PHYSICS_AUDIT_TRAIL。

### 1.2 熱力學第二定律 — 熵增原理

**定律陳述：** 孤立系統的熵永不減少。

**克勞修斯不等式：**

$$\oint \frac{\delta Q}{T} \leq 0$$

**熵增原理：**

$$dS \geq \frac{\delta Q}{T}$$

其中等號成立於可逆過程。

**微觀解釋（玻爾茲曼公式）：**

$$S = k_B \ln \Omega$$

其中 $\Omega$ 為系統的微觀態數。

```python
class SecondLawOfThermodynamics:
    """
    熱力學第二定律 - 熵增原理
    """
    
    BOLTZMANN_CONSTANT = 1.380649e-23  # J/K
    
    def __init__(self):
        self.entropy = 0.0
    
    def boltzmann_entropy(self, microstates: int) -> float:
        """
        玻爾茲曼熵公式
        
        S = k_B ln Ω
        """
        return self.BOLTZMANN_CONSTANT * np.log(microstates)
    
    def entropy_change(
        self,
        heat_transfer: float,
        temperature: float,
        reversible: bool = False
    ) -> float:
        """
        熵變計算
        
        dS ≥ δQ/T
        """
        if reversible:
            return heat_transfer / temperature
        else:
            # 不可逆過程：熵產生
            return heat_transfer / temperature
    
    def entropy_production(
        self,
        initial_entropy: float,
        final_entropy: float
    ) -> float:
        """
        計算熵產生
        
        Σ = S_f - S_i
        """
        return final_entropy - initial_entropy
    
    def carnot_efficiency(
        self,
        hot_temperature: float,
        cold_temperature: float
    ) -> float:
        """
        卡諾效率
        
        η = 1 - Tc/Th
        """
        if hot_temperature <= cold_temperature:
            raise ValueError("熱源溫度必須大於冷源溫度")
        return 1.0 - (cold_temperature / hot_temperature)
```

### 1.3 熱力學第三定律 — 絕對零度不可達

**定律陳述：** 當系統趨於絕對零度時，系統的熵趨於一個常數值（通常取為零）。

**數學表達：**

$$\lim_{T \to 0} S = S_0$$

$$\lim_{T \to 0} C = 0$$

其中 $C$ 為熱容。

**能斯特定理：**

$$T = 0 \text{ 不可達}$$

```python
class ThirdLawOfThermodynamics:
    """
    熱力學第三定律 - 絕對零度不可達
    """
    
    ABSOLUTE_ZERO = 0.0  # K
    
    def __init__(self):
        self.reference_entropy = 0.0  # 取零作為參考
    
    def entropy_at_low_temp(
        self,
        degeneracy: int,
        temperature: float,
        boltzmann: float = 1.380649e-23
    ) -> float:
        """
        低溫熵
        
        S ≈ k_B ln(g) 當 T → 0
        """
        if temperature <= 0:
            raise ValueError("溫度必須為正")
        return boltzmann * np.log(degeneracy)
    
    def heat_capacity_limit(
        self,
        temperature: float,
        low_temp_coefficient: float
    ) -> float:
        """
        熱容趨於零
        
        C → 0 當 T → 0
        """
        return low_temp_coefficient * temperature**3
    
    def verify_absolute_zero_unreachable(
        self,
        proposed_temp: float
    ) -> dict:
        """
        驗證絕對零度不可達原則
        """
        if proposed_temp <= 0:
            return {
                'valid': False,
                'message': '提議溫度低於或等於零度違反第三定律'
            }
        return {
            'valid': True,
            'message': f'溫度 { proposed_temp }K 符合第三定律'
        }
```

---

## §2. 熱力學勢函數

### 2.1 內能

內能是系統總能量的巨觀表示：

$$U = U(S, V, N)$$

**全微分：**

$$dU = TdS - PdV + \mu dN$$

```python
class InternalEnergy:
    """
    內能
    """
    
    def __init__(self, energy: float):
        self.U = energy
    
    def total_differential(
        self,
        temperature: float,
        entropy_change: float,
        pressure: float,
        volume_change: float,
        chemical_potential: float,
        particle_change: float
    ) -> float:
        """
        內能全微分
        
        dU = TdS - PdV + μdN
        """
        return (temperature * entropy_change - 
                pressure * volume_change + 
                chemical_potential * particle_change)
```

### 2.2 焓

**定義：**

$$H = U + PV$$

**物理意義：** 在常壓過程中，焓代表系統總熱含量。

**全微分：**

$$dH = TdS + VdP + \mu dN$$

```python
class Enthalpy:
    """
    焓 H = U + PV
    """
    
    def __init__(self, internal_energy: float, pressure: float, volume: float):
        self.H = internal_energy + pressure * volume
    
    def at_constant_pressure(
        self,
        heat_capacity: float,
        temp_initial: float,
        temp_final: float
    ) -> float:
        """
        定壓過程中的焓變
        
        ΔH = Cp ΔT
        """
        return heat_capacity * (temp_final - temp_initial)
    
    def reaction_enthalpy(
        self,
        reactants_enthalpy: float,
        products_enthalpy: float
    ) -> float:
        """
        反應焓
        
        ΔH = H_產物 - H_反應物
        """
        return products_enthalpy - reactants_enthalpy
```

### 2.3 亥姆霍茲自由能

**定義：**

$$F = U - TS$$

**物理意義：** 在定溫定容條件下，自發過程趨向於自由能減少。

**全微分：**

$$dF = -SdT - PdV + \mu dN$$

```python
class HelmholtzFreeEnergy:
    """
    亥姆霍茲自由能 F = U - TS
    """
    
    def __init__(self, internal_energy: float, temperature: float, entropy: float):
        self.F = internal_energy - temperature * entropy
    
    def equilibrium_condition(
        self,
        temp_initial: float,
        temp_final: float,
        vol_initial: float,
        vol_final: float,
        pressure: float
    ) -> bool:
        """
        平衡條件
        
        dF = 0
        """
        return (temp_initial == temp_final and 
                abs(pressure * (vol_final - vol_initial)) < 1e-10)
    
    def isothermal_compression_work(
        self,
        temperature: float,
        initial_volume: float,
        final_volume: float,
        n_moles: float,
        R: float = 8.314
    ) -> float:
        """
        等溫壓縮功
        
        W = -nRT ln(V2/V1)
        """
        return -n_moles * R * temperature * np.log(final_volume / initial_volume)
```

### 2.4 吉布斯自由能

**定義：**

$$G = H - TS = U + PV - TS$$

**物理意義：** 在定溫定壓條件下，自發過程趨向於吉布斯自由能減少。

**全微分：**

$$dG = -SdT + VdP + \mu dN$$

```python
class GibbsFreeEnergy:
    """
    吉布斯自由能 G = H - TS
    """
    
    def __init__(self, enthalpy: float, temperature: float, entropy: float):
        self.G = enthalpy - temperature * entropy
    
    def equilibrium_constant(
        self,
        temperature: float,
        delta_G: float,
        R: float = 8.314
    ) -> float:
        """
        化學平衡常數
        
        K = exp(-ΔG°/RT)
        """
        return np.exp(-delta_G / (R * temperature))
    
    def phase_equilibrium(
        self,
        phase1_chemical_potential: float,
        phase2_chemical_potential: float
    ) -> bool:
        """
        相平衡條件
        
        μ1 = μ2
        """
        return abs(phase1_chemical_potential - phase2_chemical_potential) < 1e-10
```

---

## §3. 熱力學過程與循環

### 3.1 基本熱力學過程

|| 過程 | 約束 | 熱力學關係 |
|-----|------|------|-----------|
| 等溫 | T = 常數 | dT = 0 | ΔU = 0 |
| 等壓 | P = 常數 | dP = 0 | ΔH = Q_p |
| 等容 | V = 常數 | dV = 0 | ΔU = Q_v |
| 絕熱 | Q = 0 | δQ = 0 | PV^γ = 常數 |
| 可逆 | 平衡 | dS = 0 | δQ_rev = TdS |

```python
class ThermodynamicProcess:
    """
    熱力學過程
    """
    
    PROCESS_TYPES = {
        'isothermal': '等溫過程 - T = 常數',
        'isobaric': '等壓過程 - P = 常數',
        'isochoric': '等容過程 - V = 常數',
        'adiabatic': '絕熱過程 - Q = 0',
        'reversible': '可逆過程 - 平衡'
    }
    
    def isothermal_work(
        self,
        n: float,
        R: float,
        T: float,
        V1: float,
        V2: float
    ) -> float:
        """
        等溫過程功
        
        W = nRT ln(V2/V1)
        """
        return n * R * T * np.log(V2 / V1)
    
    def adiabatic_relation(
        self,
        gamma: float,
        V1: float,
        V2: float,
        P1: float
    ) -> float:
        """
        絕熱方程
        
        P1*V1^γ = P2*V2^γ
        """
        return P1 * (V1 / V2)**gamma
    
    def polytropic_process(
        self,
        n: float,
        R: float,
        T1: float,
        V1: float,
        V2: float,
        index: float
    ) -> float:
        """
        多方過程
        
        PV^n = 常數
        """
        return (n * R * (T2_from_T1_V(T1, V1, V2, index) - T1) / 
                (1 - index))
```

### 3.2 熱力學循環

**循環效率：**

$$\eta = \frac{W_{net}}{Q_H} = 1 - \frac{Q_C}{Q_H}$$

```python
class ThermodynamicCycle:
    """
    熱力學循環
    """
    
    def thermal_efficiency(
        self,
        heat_input: float,
        heat_output: float
    ) -> float:
        """
        熱效率
        
        η = 1 - Qc/Qh
        """
        return 1.0 - (heat_output / heat_input)
    
    def coefficient_of_performance(
        self,
        heat_dissipated: float,
        work_input: float
    ) -> float:
        """
        製冷係數
        
        COP = Qc/W
        """
        return heat_dissipated / work_input
    
    def carnot_efficiency(
        self,
        T_hot: float,
        T_cold: float
    ) -> float:
        """
        卡諾效率
        
        η = 1 - Tc/Th
        """
        return 1.0 - (T_cold / T_hot)
```

---

## §4. 卡諾循環與效率

### 4.1 卡諾循環

卡諾循環是理想的可逆熱力學循環，由四個可逆過程組成：

1. **等溫膨脹**：從高溫熱源吸熱 $Q_H$
2. **絕熱膨脹**：溫度降至 $T_C$
3. **等溫壓縮**：向低溫熱源放熱 $Q_C$
4. **絕熱壓縮**：溫度回升至 $T_H$

**數學表達：**

$$Q_H = nRT_H \ln\left(\frac{V_2}{V_1}\right)$$

$$Q_C = nRT_C \ln\left(\frac{V_4}{V_3}\right)$$

$$\eta_{Carnot} = 1 - \frac{T_C}{T_H}$$

```python
class CarnotCycle:
    """
    卡諾循環 - 理想可逆熱機
    """
    
    def __init__(self, T_hot: float, T_cold: float):
        self.T_hot = T_hot  # K
        self.T_cold = T_cold  # K
    
    def efficiency(self) -> float:
        """
        卡諾效率
        
        η = 1 - Tc/Th
        """
        return 1.0 - (self.T_cold / self.T_hot)
    
    def heat_absorbed(
        self,
        n_moles: float,
        R: float,
        volume_ratio: float
    ) -> float:
        """
        吸熱量
        
        Qh = nRTc ln(V2/V1)
        """
        return n_moles * R * self.T_hot * np.log(volume_ratio)
    
    def heat_rejected(
        self,
        n_moles: float,
        R: float,
        volume_ratio: float
    ) -> float:
        """
        放熱量
        
        Qc = nRTc ln(V4/V3)
        """
        return n_moles * R * self.T_cold * np.log(volume_ratio)
    
    def work_done(
        self,
        heat_absorbed: float
    ) -> float:
        """
        淨功
        
        W = Qh - Qc = η * Qh
        """
        return heat_absorbed * self.efficiency()
    
    def cop_refrigerator(self) -> float:
        """
        卡諾製冷機 COP
        
        COP = Tc/(Th - Tc)
        """
        return self.T_cold / (self.T_hot - self.t_cold)
    
    def cop_heat_pump(self) -> float:
        """
        卡諾熱泵 COP
        
        COP = Th/(Th - Tc)
        """
        return self.T_hot / (self.T_hot - self.T_cold)
```

### 4.2 真實熱機的限制

**實際效率：**

$$\eta_{real} = \eta_{Carnot} \cdot \eta_{mechanical}$$

**不可逆性損失：**

$$\eta_{real} = 1 - \frac{T_C}{T_H} - \Sigma_i$$

其中 $\Sigma_i$ 為各不可逆過程的熵產生。

```python
class RealHeatEngine:
    """
    真實熱機
    """
    
    def __init__(self):
        self.irreversibilities = []
    
    def real_efficiency(
        self,
        carnot_efficiency: float,
        mechanical_efficiency: float = 0.9
    ) -> float:
        """
        實際效率
        
        η_real = η_Carnot × η_mech × (1 - Σ)
        """
        return carnot_efficiency * mechanical_efficiency
    
    def add_irreversibility(
        self,
        name: str,
        entropy_production: float
    ) -> None:
        """
        添加不可逆性
        """
        self.irreversibilities.append({
            'name': name,
            'entropy': entropy_production
        })
    
    def total_entropy_production(self) -> float:
        """
        總熵產生
        """
        return sum(item['entropy'] for item in self.irreversibilities)
```

---

## §5. 與統計力學的接口

### 5.1 巨觀-微觀橋樑

熱力學與統計力學通過以下對應關係連接：

| 巨觀熱力學量 | 統計力學表達 |
|------------|-------------|
| $S$ | $k_B \ln \Omega$ |
| $T$ | $\partial U / \partial S$ |
| $P$ | $-\partial U / \partial V$ |
| $\mu$ | $\partial U / \partial N$ |

### 5.2 從配分函數到熱力學

```python
class ThermodynamicsFromStatistics:
    """
    從統計力學推導熱力學量
    """
    
    def __init__(self, partition_function: float, temperature: float):
        self.Z = partition_function
        self.T = temperature
        self.beta = 1.0 / (1.380649e-23 * temperature)
    
    def free_energy(self) -> float:
        """
        亥姆霍茲自由能
        
        F = -k_B T ln Z
        """
        return -1.380649e-23 * self.T * np.log(self.Z)
    
    def entropy(self) -> float:
        """
        熵
        
        S = k_B (ln Z + βU)
        """
        U = self.internal_energy()
        return 1.380649e-23 * (np.log(self.Z) + self.beta * U)
    
    def internal_energy(self) -> float:
        """
        內能
        
        U = -∂ ln Z / ∂β
        """
        return -np.log(self.Z) / self.beta
    
    def pressure(self) -> float:
        """
        壓力
        
        P = k_B T ∂ ln Z / ∂V
        """
        return 1.380649e-23 * self.T * np.log(self.Z)  # 簡化示例
```

### 5.3 漲落與熱力學極限

在熱力學極限下（$N \to \infty$），相對漲落趨於零：

$$\frac{\langle (\Delta A)^2 \rangle}{\langle A \rangle^2} \propto \frac{1}{\sqrt{N}}$$

---

## §6. 前沿研究進展

### 6.1 量子熱力學的最新進展

**熱力學前沿研究：**

|| 研究領域 | 進展 | 意義 |
|---------|---------|------|------|
| 量子熱機 | 單光子水平的熱功轉換 | 探索量子效應對熱機效率的影響 |
| 量子冰箱 | 基於量子相變的製冷機制 | 極低溫製冷新方案 |
| 資訊熱力學 | 蘭道爾極限的量子修正 | 量子資訊處理的熱力學基礎 |

### 6.2 機器學習與熱力學

**AI輔助熱力學研究：**

```python
class MachineLearningThermodynamics:
    """
    機器學習輔助熱力學
    """
    
    RECENT_APPLICATIONS = {
        'phase_diagram_prediction': '深度學習預測相圖',
        'equation_of_state': '神經網路擬合狀態方程',
        'entropy_calculation': 'ML加速熵計算',
        'material_discovery': '逆設計熱電材料'
    }
    
    def predict_phase_boundary(
        self,
        model,
        temperature_range: tuple,
        pressure_range: tuple
    ) -> np.ndarray:
        """
        預測相邊界
        """
        pass  # 實現依賴具體模型
```

> **真理協議提醒：** 上述前沿研究進展基於公開研究。所有具體數值和結論應通過原始文獻驗證。

---

## 版本歷史

| 版本 | 日期 | 變更描述 | 作者 |
|------|------|---------|------|
| v1.0 | 2026-03-18 | 初始版本 - 熱力學完整模組 | NoieAGENTS |

---

*本文檔處理巨觀熱力學尺度的物理框架。*
*熱力學連接微觀與巨觀，是物理學的重要橋樑。*
*與 STATISTICAL_MECHANICS.md 共同構成完整的統計熱力學體系。*
