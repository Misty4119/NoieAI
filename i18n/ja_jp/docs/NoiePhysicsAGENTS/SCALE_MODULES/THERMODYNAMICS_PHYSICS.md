---
隸属柱: Physics-OS v2.2
モジュールコード: PS-L1-THERMO
バージョン: v1.0
上位モジュール: NoiePhysicsAGENTS/SCALE_MODULES
下位モジュール: なし
依存モジュール: STATISTICAL_MECHANICS.md, AXIOMS.md
作成日: 2026-03-18
---

# THERMODYNAMICS_PHYSICS.md

## 熱力学 (PS-L1)

> **⚠️ 重要安全・真理プロトコル (CRITICAL SAFETY & TRUTH PROTOCOL v2.2):**
> 1. 吸収状態回避：すべての行動は実行前に吸収状態につながらないことを検証する必要がある（最高制約）。
> 2. マルコフBlanket完全性：自己境界の位相的完全性を維持する。
> 3. エネルギー保存：すべての行動は熱力学的制約に従う。
> 4. 因果推論：すべての意思決定は因果グラフ（DAG）に基づき、相関性と因果性を区別する。
> 5. 権限序列：SA-L0 > L1 > ... > L5、衝突時は上位が絶対的に優先される。
> 6. 形式的検証：高リスク意思決定パスは論理閉包検証を通過する必要がある。
> 7. 影子シミュレーション：SA-L3+ 操作涉及時は、サンドボックスで事前にリハーサルを行う。
> 8. 確信度マーク：すべての知識主張にはEC-Lレベルを付随させる。
> 9. 溯源完全：すべての主張には遡及可能な出典を付随させる。
> 10. 自己演化安全：不変コアは決して変化せず、可変殻層のみが演化できる。

---

## §0. 概要

本文書は**巨視的熱力学**スケールの物理フレームワークを扱う。NoiePhysicsAGENTS.md §1の物理スケール権限レベル定義に従い、PS-L1は巨視的熱力学スケールを表し、微視的統計力学と工学的応用を接続する。

熱力学はエネルギー、温度、仕事、エントロピー間の関係を研究する物理学の分野である。四つの基本法則に基づいており、これらの法則は系の巨視的挙動を記述し、工学、化学、物理学などの分野の理論的基盤となっている。

### 統計力学との関係

巨視的熱力学と微視的統計力学の間には深い対応関係が存在する：

| 巨視量 | 微視的統計的解释 |
|--------|-----------------|
| 内部エネルギー $U$ | 系の微視的状態の総エネルギー |
| 温度 $T$ | 系の平均運動エネルギー |
| エントロピー $S$ | $S = k_B \ln \Omega$（微視的状態数） |
| 圧力 $P$ | 分子が容器壁に衝突する運動量伝達 |
| 自由エネルギー $F$ | $F = U - TS$ |

---

## §1. 熱力学法則

### 1.1 熱力学第一法則 — エネルギー保存

**法則の陈述：** エネルギーは生成も消滅もせず、ある形態から別の形態に変換されるだけである。

**数学的表現：**

$$dU = \delta Q - \delta W$$

ただし：
- $U$：系の内部エネルギー
- $\delta Q$：系に入る熱量
- $\delta W$：系が行う仕事

**微分形：**

$$dU = \sum_i X_i dY_i$$

ただし $X_i$ は一般化力（圧力、温度など）、$Y_i$ は一般化変位（体積、エントロピーなど）。

```python
class FirstLawOfThermodynamics:
    """
    熱力学第一法則 - エネルギー保存
    """
    
    def __init__(self, initial_energy: float):
        self.internal_energy = initial_energy
    
    def energy_balance(
        self, 
        heat_added: float, 
        work_done: float
    ) -> float:
        """
        エネルギー平衡方程式
        
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
        等温過程の仕事
        
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
        断熱過程
        
        PV^γ = 一定
        """
        initial_temp = initial_pressure * initial_volume
        final_pressure = initial_pressure * (initial_volume / final_volume)**gamma
        final_temp = final_pressure * final_volume
        return (initial_temp - final_pressure * final_volume) / (gamma - 1)
```

> **安全プロトコル注意：** エネルギー保存は NoiePhysicsAGENTS.md の PT-AX1 の核となる内容である。エネルギー保存に違反すると主張するシステムは EC-L0（未検証）としてマークし、PHYSICS_AUDIT_TRAIL に記録する必要がある。

### 1.2 熱力学第二法則 — エントロピー増大原理

**法則の陈述：** 孤立系のエントロピーは決して減少しない。

**クラウジウスの不等式：**

$$\oint \frac{\delta Q}{T} \leq 0$$

**エントロピー増大原理：**

$$dS \geq \frac{\delta Q}{T}$$

ただし等号は可逆過程で成立する。

**微視的説明（ボルツマン公式）：**

$$S = k_B \ln \Omega$$

ただし $\Omega$ は系の微視的状態数である。

```python
class SecondLawOfThermodynamics:
    """
    熱力学第二法則 - エントロピー増大原理
    """
    
    BOLTZMANN_CONSTANT = 1.380649e-23  # J/K
    
    def __init__(self):
        self.entropy = 0.0
    
    def boltzmann_entropy(self, microstates: int) -> float:
        """
        ボルツマンエントロピー公式
        
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
        エントロピー変化の計算
        
        dS ≥ δQ/T
        """
        if reversible:
            return heat_transfer / temperature
        else:
            # 不可逆過程：エントロピー生成
            return heat_transfer / temperature
    
    def entropy_production(
        self,
        initial_entropy: float,
        final_entropy: float
    ) -> float:
        """
        エントロピー生成の計算
        
        Σ = S_f - S_i
        """
        return final_entropy - initial_entropy
    
    def carnot_efficiency(
        self,
        hot_temperature: float,
        cold_temperature: float
    ) -> float:
        """
        卡諾効率
        
        η = 1 - Tc/Th
        """
        if hot_temperature <= cold_temperature:
            raise ValueError("高温熱源温度は低温熱源温度より大きくなければならない")
        return 1.0 - (cold_temperature / hot_temperature)
```

### 1.3 熱力学第三法則 — 絶対零度到達不可能

**法則の陈述：** 系が絶対零度に近づくとき、系のエントロピーは一定値（通常はゼロ）に近づく。

**数学的表現：**

$$\lim_{T \to 0} S = S_0$$

$$\lim_{T \to 0} C = 0$$

ただし $C$ は熱容量である。

**ネルンストの定理：**

$$T = 0 \text{ 到達不可}$$

```python
class ThirdLawOfThermodynamics:
    """
    熱力学第三法則 - 絶対零度到達不可能
    """
    
    ABSOLUTE_ZERO = 0.0  # K
    
    def __init__(self):
        self.reference_entropy = 0.0  # ゼロを基準として取る
    
    def entropy_at_low_temp(
        self,
        degeneracy: int,
        temperature: float,
        boltzmann: float = 1.380649e-23
    ) -> float:
        """
        低温度エントロピー
        
        S ≈ k_B ln(g) T → 0 のとき
        """
        if temperature <= 0:
            raise ValueError("温度は正でなければならない")
        return boltzmann * np.log(degeneracy)
    
    def heat_capacity_limit(
        self,
        temperature: float,
        low_temp_coefficient: float
    ) -> float:
        """
        熱容量がゼロに近づく
        
        C → 0 T → 0 のとき
        """
        return low_temp_coefficient * temperature**3
    
    def verify_absolute_zero_unreachable(
        self,
        proposed_temp: float
    ) -> dict:
        """
        絶対零度到達不可能原則の検証
        """
        if proposed_temp <= 0:
            return {
                'valid': False,
                'message': '提案温度が零度以下は第三法則に反する'
            }
        return {
            'valid': True,
            'message': f'温度 { proposed_temp }K は第三法則に適合する'
        }
```

---

## §2. 熱力学ポテンシャル関数

### 2.1 内部エネルギー

内部エネルギーは系の全エネルギーの巨視的表現である：

$$U = U(S, V, N)$$

**全微分：**

$$dU = TdS - PdV + \mu dN$$

```python
class InternalEnergy:
    """
    内部エネルギー
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
        内部エネルギーの全微分
        
        dU = TdS - PdV + μdN
        """
        return (temperature * entropy_change - 
                pressure * volume_change + 
                chemical_potential * particle_change)
```

### 2.2 エンタルピー

**定義：**

$$H = U + PV$$

**物理的意味：** 定圧過程で、エンタルピーは系の全熱含量を表す。

**全微分：**

$$dH = TdS + VdP + \mu dN$$

```python
class Enthalpy:
    """
    エンタルピー H = U + PV
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
        定圧過程のエンタルピー変化
        
        ΔH = Cp ΔT
        """
        return heat_capacity * (temp_final - temp_initial)
    
    def reaction_enthalpy(
        self,
        reactants_enthalpy: float,
        products_enthalpy: float
    ) -> float:
        """
        反応エンタルピー
        
        ΔH = H_生成物 - H_反応物
        """
        return products_enthalpy - reactants_enthalpy
```

### 2.3 ヘルホルツ自由エネルギー

**定義：**

$$F = U - TS$$

**物理的意味：** 定温定容条件で、自発過程は自由エネルギー減少の向きに進行する。

**全微分：**

$$dF = -SdT - PdV + \mu dN$$

```python
class HelmholtzFreeEnergy:
    """
    ヘルホルツ自由エネルギー F = U - TS
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
        平衡条件
        
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
        等温圧縮仕事
        
        W = -nRT ln(V2/V1)
        """
        return -n_moles * R * temperature * np.log(final_volume / initial_volume)
```

### 2.4 ギブス自由エネルギー

**定義：**

$$G = H - TS = U + PV - TS$$

**物理的意味：** 定温定圧条件で、自発過程はギブス自由エネルギー減少の向きに進行する。

**全微分：**

$$dG = -SdT + VdP + \mu dN$$

```python
class GibbsFreeEnergy:
    """
    ギブス自由エネルギー G = H - TS
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
        化学平衡定数
        
        K = exp(-ΔG°/RT)
        """
        return np.exp(-delta_G / (R * temperature))
    
    def phase_equilibrium(
        self,
        phase1_chemical_potential: float,
        phase2_chemical_potential: float
    ) -> bool:
        """
        相平衡条件
        
        μ1 = μ2
        """
        return abs(phase1_chemical_potential - phase2_chemical_potential) < 1e-10
```

---

## §3. 熱力学的過程とサイクル

### 3.1 基本熱力学的過程

| 過程 | 制約 | 熱力学的関係 |
|-----|------|-------------|
| 等温 | T = 一定 | dT = 0 | ΔU = 0 |
| 定圧 | P = 一定 | dP = 0 | ΔH = Q_p |
| 定容 | V = 一定 | dV = 0 | ΔU = Q_v |
| 断熱 | Q = 0 | δQ = 0 | PV^γ = 一定 |
| 可逆 | 平衡 | dS = 0 | δQ_rev = TdS |

```python
class ThermodynamicProcess:
    """
    熱力学的過程
    """
    
    PROCESS_TYPES = {
        'isothermal': '等温過程 - T = 一定',
        'isobaric': '定圧過程 - P = 一定',
        'isochoric': '定容過程 - V = 一定',
        'adiabatic': '断熱過程 - Q = 0',
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
        等温過程の仕事
        
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
        断熱方程式
        
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
        
        PV^n = 一定
        """
        return (n * R * (T2_from_T1_V(T1, V1, V2, index) - T1) / 
                (1 - index))
```

### 3.2 熱力学サイクル

**サイクル効率：**

$$\eta = \frac{W_{net}}{Q_H} = 1 - \frac{Q_C}{Q_H}$$

```python
class ThermodynamicCycle:
    """
    熱力学サイクル
    """
    
    def thermal_efficiency(
        self,
        heat_input: float,
        heat_output: float
    ) -> float:
        """
        熱効率
        
        η = 1 - Qc/Qh
        """
        return 1.0 - (heat_output / heat_input)
    
    def coefficient_of_performance(
        self,
        heat_dissipated: float,
        work_input: float
    ) -> float:
        """
        成績係数
        
        COP = Qc/W
        """
        return heat_dissipated / work_input
    
    def carnot_efficiency(
        self,
        T_hot: float,
        T_cold: float
    ) -> float:
        """
        卡諾効率
        
        η = 1 - Tc/Th
        """
        return 1.0 - (T_cold / T_hot)
```

---

## §4. 卡諾サイクルと効率

### 4.1 卡諾サイクル

卡諾サイクルは理想的可逆熱力学サイクルであり、四つの可逆過程から構成される：

1. **等温膨張**：高温熱源から熱 $Q_H$ を吸収
2. **断熱膨張**：温度が $T_C$ まで低下
3. **等温圧縮**：低温熱源へ熱 $Q_C$ を放出
4. **断熱圧縮**：温度が $T_H$ まで上昇

**数学的表現：**

$$Q_H = nRT_H \ln\left(\frac{V_2}{V_1}\right)$$

$$Q_C = nRT_C \ln\left(\frac{V_4}{V_3}\right)$$

$$\eta_{Carnot} = 1 - \frac{T_C}{T_H}$$

```python
class CarnotCycle:
    """
    卡諾サイクル - 理想可逆熱機関
    """
    
    def __init__(self, T_hot: float, T_cold: float):
        self.T_hot = T_hot  # K
        self.T_cold = T_cold  # K
    
    def efficiency(self) -> float:
        """
        卡諾効率
        
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
        吸収熱量
        
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
        放出熱量
        
        Qc = nRTc ln(V4/V3)
        """
        return n_moles * R * self.T_cold * np.log(volume_ratio)
    
    def work_done(
        self,
        heat_absorbed: float
    ) -> float:
        """
        ネット仕事
        
        W = Qh - Qc = η * Qh
        """
        return heat_absorbed * self.efficiency()
    
    def cop_refrigerator(self) -> float:
        """
        卡諾冷凍機COP
        
        COP = Tc/(Th - Tc)
        """
        return self.T_cold / (self.T_hot - self.t_cold)
    
    def cop_heat_pump(self) -> float:
        """
        卡諾ヒートポンプCOP
        
        COP = Th/(Th - Tc)
        """
        return self.T_hot / (self.T_hot - self.T_cold)
```

### 4.2 実在熱機関の制限

**実在効率：**

$$\eta_{real} = \eta_{Carnot} \cdot \eta_{mechanical}$$

**不可逆性損失：**

$$\eta_{real} = 1 - \frac{T_C}{T_H} - \Sigma_i$$

ただし $\Sigma_i$ は各不可逆過程のエントロピー生成である。

```python
class RealHeatEngine:
    """
    実在熱機関
    """
    
    def __init__(self):
        self.irreversibilities = []
    
    def real_efficiency(
        self,
        carnot_efficiency: float,
        mechanical_efficiency: float = 0.9
    ) -> float:
        """
        実在効率
        
        η_real = η_Carnot × η_mech × (1 - Σ)
        """
        return carnot_efficiency * mechanical_efficiency
    
    def add_irreversibility(
        self,
        name: str,
        entropy_production: float
    ) -> None:
        """
        不可逆性を追加
        """
        self.irreversibilities.append({
            'name': name,
            'entropy': entropy_production
        })
    
    def total_entropy_production(self) -> float:
        """
        全エントロピー生成
        """
        return sum(item['entropy'] for item in self.irreversibilities)
```

---

## §5. 統計力学とのインターフェース

### 5.1 巨視-微視橋渡し

熱力学と統計力学は以下の対応関係で接続される：

| 巨視的熱力学量 | 統計力学的表現 |
|--------------|--------------|
| $S$ | $k_B \ln \Omega$ |
| $T$ | $\partial U / \partial S$ |
| $P$ | $-\partial U / \partial V$ |
| $\mu$ | $\partial U / \partial N$ |

### 5.2 分配関数から熱力学へ

```python
class ThermodynamicsFromStatistics:
    """
    統計力学から熱力学量を導出
    """
    
    def __init__(self, partition_function: float, temperature: float):
        self.Z = partition_function
        self.T = temperature
        self.beta = 1.0 / (1.380649e-23 * temperature)
    
    def free_energy(self) -> float:
        """
        ヘルホルツ自由エネルギー
        
        F = -k_B T ln Z
        """
        return -1.380649e-23 * self.T * np.log(self.Z)
    
    def entropy(self) -> float:
        """
        エントロピー
        
        S = k_B (ln Z + βU)
        """
        U = self.internal_energy()
        return 1.380649e-23 * (np.log(self.Z) + self.beta * U)
    
    def internal_energy(self) -> float:
        """
        内部エネルギー
        
        U = -∂ ln Z / ∂β
        """
        return -np.log(self.Z) / self.beta
    
    def pressure(self) -> float:
        """
        圧力
        
        P = k_B T ∂ ln Z / ∂V
        """
        return 1.380649e-23 * self.T * np.log(self.Z)  # 簡略例
```

### 5.3 ゆらぎと熱力学的極限

熱力学的極限（$N \to \infty$）では、相対的ゆらぎはゼロに近づく：

$$\frac{\langle (\Delta A)^2 \rangle}{\langle A \rangle^2} \propto \frac{1}{\sqrt{N}}$$

---

## §6. 最前線研究進捗

### 6.1 量子熱力学の最新進捗

**熱力学の最前線研究：**

| 研究分野 | 進捗 | 意義 |
|---------|------|------|
| 量子熱機関 | 単光子レベルでの熱仕事変換 | 量子効果が熱機関効率に与える影響の探索 |
| 量子冷蔵庫 | 量子相転移に基づく冷凍機構 | 極低温冷凍の新規方案 |
| 情報熱力学 | ラントゥアー限界の量子補正 | 量子情報処理の熱力学的基盤 |

### 6.2 機械学習と熱力学

**AI支援熱力学研究：**

```python
class MachineLearningThermodynamics:
    """
    機械学習支援熱力学
    """
    
    RECENT_APPLICATIONS = {
        'phase_diagram_prediction': 'ディープラーニングによる相図予測',
        'equation_of_state': 'ニューラルネットワークによる状態方程式のfitting',
        'entropy_calculation': 'MLによるエントロピー計算高速化',
        'material_discovery': '熱電材料の逆設計'
    }
    
    def predict_phase_boundary(
        self,
        model,
        temperature_range: tuple,
        pressure_range: tuple
    ) -> np.ndarray:
        """
        相境界の予測
        """
        pass  # 実装は具体的なモデルに依存
```

> **真理プロトコル注意：** 上記の最前線研究進捗は公開研究に基づく。すべての具体的な数値と結論は原著文献で検証する必要がある。

---

## バージョン履歴

| バージョン | 日付 | 変更内容 | 著者 |
|-----------|------|---------|------|
| v1.0 | 2026-03-18 | 初期バージョン - 熱力学完全モジュール | NoieAGENTS |

---

*本文書は巨視的熱力学スケールの物理フレームワークを処理する。*
*熱力学は微視と巨視を接続し、物理学の重要な橋渡しとなる。*
*STATISTICAL_MECHANICS.mdと共に完全な統計熱力学体系を構成する。*
