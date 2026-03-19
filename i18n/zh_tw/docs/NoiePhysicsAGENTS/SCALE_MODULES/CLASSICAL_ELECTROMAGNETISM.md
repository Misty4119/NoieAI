# CLASSICAL_ELECTROMAGNETISM.md

## 經典電磁學 (PS-L2)

**尺度：** 10⁻⁹ ~ 10⁷ m  
**版本：** v1.0  
**狀態：** 驗證性

---

## 概述

本文檔處理**經典電磁學**的物理框架。根據 NoiePhysicsAGENTS.md §1 的物理尺度權限層級定義，PS-L2 代表古典尺度，涵蓋從微觀粒子到巨觀物體的电磁现象。

經典電磁學是物理學最成功和經過最充分驗證的理論之一，其數學結構優美且預測精確。

---

## 關鍵安全與真理協議

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

## 1. 靜電學 (Electrostatics)

### 1.1 庫侖定律

兩個點電荷之間的電力：

$$\vec{F} = \frac{1}{4\pi\varepsilon_0} \frac{q_1 q_2}{r^2} \hat{r}$$

```python
class CoulombLaw:
    """
    庫侖定律
    """
    
    COULOMB_CONSTANT = 8.9875517923e9  # N·m²/C²
    
    def electric_force(
        self,
        q1: float,
        q2: float,
        r: float
    ) -> float:
        """計算兩個點電荷之間的電力大小"""
        # F = k * |q1 * q2| / r²
        return self.COULOMB_CONSTANT * abs(q1 * q2) / r**2
    
    def electric_field(
        self,
        q: float,
        r: float
    ) -> float:
        """計算點電荷產生的電場"""
        # E = k * |q| / r²
        return self.COULOMB_CONSTANT * abs(q) / r**2
```

### 1.2 電場與電位

電場是電荷周圍的向量場：

$$\vec{E} = -\nabla V$$

電位與電場的關係：

$$V(\vec{r}) = \frac{1}{4\pi\varepsilon_0} \int \frac{\rho(\vec{r}')}{|\vec{r} - \vec{r}'|} d^3r'$$

```python
class ElectricField:
    """
    電場計算
    """
    
    def point_charge_field(
        self,
        charge: float,
        position: np.ndarray,
        observation_point: np.ndarray
    ) -> np.ndarray:
        """計算點電荷在某點產生的電場"""
        r = observation_point - position
        distance = np.linalg.norm(r)
        direction = r / distance
        magnitude = self.COULOMB_CONSTANT * abs(charge) / distance**2
        return magnitude * direction
    
    def superposition(
        self,
        charges: List[Charge],
        observation_point: np.ndarray
    ) -> np.ndarray:
        """疊加原理：多個電荷的電場"""
        total_field = np.zeros(3)
        for charge in charges:
            total_field += self.point_charge_field(
                charge.q, charge.position, observation_point
            )
        return total_field
```

### 1.3 高斯定律

電場通過任意閉曲面的通量與內部電荷有關：

$$\oint_S \vec{E} \cdot d\vec{A} = \frac{Q_{\text{enc}}}{\varepsilon_0}$$

```python
class GaussLaw:
    """
    高斯定律
    """
    
    def electric_flux(
        self,
        electric_field: np.ndarray,
        surface: Surface
    ) -> float:
        """計算電場通過曲面的通量"""
        # Φ_E = ∮ E · dA
        return np.sum(electric_field * surface.area_elements)
    
    def enclosed_charge(
        self,
        flux: float
    ) -> float:
        """從通量計算內部電荷"""
        # Q = Φ * ε₀
        return flux * constants.epsilon_0
```

### 1.4 電位能

電荷在電場中的位能：

$$U = qV = \frac{1}{4\pi\varepsilon_0} \frac{q_1 q_2}{r}$$

```python
class ElectricPotentialEnergy:
    """
    電位能
    """
    
    def system_energy(
        self,
        charges: List[Charge]
    ) -> float:
        """計算多電荷系統的總電位能"""
        total_energy = 0.0
        n = len(charges)
        for i in range(n):
            for j in range(i + 1, n):
                r = np.linalg.norm(charges[i].position - charges[j].position)
                energy = self.COULOMB_CONSTANT * charges[i].q * charges[j].q / r
                total_energy += energy
        return total_energy
```

---

## 2. 靜磁學 (Magnetostatics)

### 2.1 必歐-沙伐定律

電流產生的磁場：

$$d\vec{B} = \frac{\mu_0}{4\pi} \frac{I d\vec{l} \times \hat{r}}{r^2}$$

```python
class BiotSavartLaw:
    """
    必歐-沙伐定律
    """
    
    MU_0 = 4 * np.pi * 1e-7  # T·m/A
    
    def magnetic_field_from_wire(
        self,
        current: float,
        wire_segment: np.ndarray,
        observation_point: np.ndarray
    ) -> np.ndarray:
        """計算直導線段在某點產生的磁場"""
        r = observation_point - wire_segment['position']
        distance = np.linalg.norm(r)
        direction = r / distance
        dl = wire_segment['direction']
        cross = np.cross(dl, direction)
        magnitude = self.MU_0 * current / (4 * np.pi * distance**2) * np.linalg.norm(cross)
        return magnitude * cross / np.linalg.norm(cross)
    
    def magnetic_field_infinite_wire(
        self,
        current: float,
        distance: float
    ) -> float:
        """無限長直導線的磁場"""
        # B = μ₀I / (2πr)
        return self.MU_0 * current / (2 * np.pi * distance)
```

### 2.2 安培定律

磁場環繞電流：

$$\oint_C \vec{B} \cdot d\vec{l} = \mu_0 I_{\text{enc}}$$

```python
class AmpereLaw:
    """
    安培定律
    """
    
    def magnetic_field_solenoid(
        self,
        n: float,
        I: float
    ) -> float:
        """理想螺線管的磁場"""
        # B = μ₀nI
        return self.MU_0 * n * I
    
    def magnetic_field_toroid(
        self,
        N: int,
        I: float,
        r: float
    ) -> float:
        """環形線圈的磁場"""
        # B = μ₀NI / (2πr)
        return self.MU_0 * N * I / (2 * np.pi * r)
```

### 2.3 洛倫茲力

運動電荷在磁場中受到的力：

$$\vec{F} = q\vec{v} \times \vec{B}$$

```python
class LorentzForce:
    """
    洛倫茲力
    """
    
    def magnetic_force(
        self,
        charge: float,
        velocity: np.ndarray,
        magnetic_field: np.ndarray
    ) -> np.ndarray:
        """計算運動電荷在磁場中受到的力"""
        return charge * np.cross(velocity, magnetic_field)
    
    def cyclotron_frequency(
        self,
        charge: float,
        mass: float,
        magnetic_field: float
    ) -> float:
        """迴旋加速頻率"""
        # ω = qB/m
        return abs(charge) * magnetic_field / mass
    
    def gyroradius(
        self,
        mass: float,
        velocity_perp: float,
        charge: float,
        magnetic_field: float
    ) -> float:
        """迴旋半徑（拉摩半徑）"""
        # r = mv_perp / (|q|B)
        return mass * velocity_perp / (abs(charge) * magnetic_field)
```

---

## 3. 麥克斯韋方程組 (Maxwell's Equations)

### 3.1 積分形式

| 定律 | 積分形式 | 意義 |
|------|---------|------|
| 高斯定律（電場） | $\oint_S \vec{E} \cdot d\vec{A} = \frac{Q}{\varepsilon_0}$ | 電荷產生電場 |
| 高斯定律（磁場） | $\oint_S \vec{B} \cdot d\vec{A} = 0$ | 無磁單極子 |
| 法拉第定律 | $\oint_C \vec{E} \cdot d\vec{l} = -\frac{d}{dt}\int_S \vec{B} \cdot d\vec{A}$ | 變化磁場產生電場 |
| 安培-馬克士威定律 | $\oint_C \vec{B} \cdot d\vec{l} = \mu_0(I + \varepsilon_0 \frac{d}{dt}\int_S \vec{E} \cdot d\vec{A})$ | 電流和變化電場產生磁場 |

### 3.2 微分形式

$$\nabla \cdot \vec{E} = \frac{\rho}{\varepsilon_0}$$

$$\nabla \cdot \vec{B} = 0$$

$$\nabla \times \vec{E} = -\frac{\partial \vec{B}}{\partial t}$$

$$\nabla \times \vec{B} = \mu_0 \vec{J} + \mu_0 \varepsilon_0 \frac{\partial \vec{E}}{\partial t}$$

```python
class MaxwellEquations:
    """
    麥克斯韋方程組
    """
    
    def gauss_law_electric(
        self,
        electric_field: Callable,
        position: np.ndarray
    ) -> float:
        """計算電場的散度"""
        # ∇ · E = ρ/ε₀
        pass
    
    def gauss_law_magnetic(
        self,
        magnetic_field: Callable,
        position: np.ndarray
    ) -> float:
        """計算磁場的散度"""
        # ∇ · B = 0
        pass
    
    def faraday_law(
        self,
        electric_field: Callable,
        time: float,
        position: np.ndarray
    ) -> np.ndarray:
        """計算法拉第定律"""
        # ∇ × E = -∂B/∂t
        pass
    
    def ampere_maxwell_law(
        self,
        magnetic_field: Callable,
        current_density: np.ndarray,
        time: float,
        position: np.ndarray
    ) -> np.ndarray:
        """計算安培-馬克士威定律"""
        # ∇ × B = μ₀J + μ₀ε₀∂E/∂t
        pass
```

### 3.3 連續性方程

電荷守恆：

$$\nabla \cdot \vec{J} + \frac{\partial \rho}{\partial t} = 0$$

```python
class ChargeConservation:
    """
    電荷守恆
    """
    
    def continuity_equation(
        self,
        current_density: Callable,
        charge_density: Callable,
        time: float
    ) -> float:
        """連續性方程：∂ρ/∂t + ∇·J = 0"""
        pass
```

---

## 4. 电磁波 (Electromagnetic Waves)

### 4.1 电磁波方程

從麥克斯韋方程組導出：

$$\nabla^2 \vec{E} - \mu_0 \varepsilon_0 \frac{\partial^2 \vec{E}}{\partial t^2} = 0$$

$$\nabla^2 \vec{B} - \mu_0 \varepsilon_0 \frac{\partial^2 \vec{B}}{\partial t^2} = 0$$

光速：

$$c = \frac{1}{\sqrt{\mu_0 \varepsilon_0}} \approx 2.998 \times 10^8 \text{ m/s}$$

```python
class ElectromagneticWave:
    """
    电磁波
    """
    
    SPEED_OF_LIGHT = 2.99792458e8  # m/s
    
    def wave_equation_electric(
        self,
        wave_vector: np.ndarray,
        omega: float
    ) -> bool:
        """驗證是否滿足波動方程"""
        # k = ω/c
        k_magnitude = omega / self.SPEED_OF_LIGHT
        return np.allclose(np.linalg.norm(wave_vector), k_magnitude)
    
    def plane_wave_electric(
        self,
        amplitude: float,
        k: np.ndarray,
        omega: float,
        position: np.ndarray,
        time: float,
        polarization: np.ndarray
    ) -> np.ndarray:
        """平面电磁波的电场"""
        # E = E₀ cos(k·r - ωt) ε̂
        phase = np.dot(k, position) - omega * time
        return amplitude * np.cos(phase) * polarization
```

### 4.2 电磁波的性質

- **橫波**：電場和磁場都垂直於傳播方向
- **偏振**：電場方向確定了偏振態
- **傳播速度**：真空中為 $c$
- **能量密度**：

$$u = \frac{1}{2}\varepsilon_0 E^2 + \frac{1}{2\mu_0} B^2 = \varepsilon_0 E^2$$

- **波印廷向量**（能量通量）：

$$\vec{S} = \frac{1}{\mu_0} \vec{E} \times \vec{B}$$

```python
class ElectromagneticEnergy:
    """
    电磁能量
    """
    
    def energy_density(
        self,
        electric_field: float,
        magnetic_field: float
    ) -> float:
        """計算电磁能量密度"""
        # u = ε₀E²/2 + B²/2μ₀
        e_term = 0.5 * constants.epsilon_0 * electric_field**2
        b_term = 0.5 * magnetic_field**2 / constants.mu_0
        return e_term + b_term
    
    def poynting_vector(
        self,
        electric_field: np.ndarray,
        magnetic_field: np.ndarray
    ) -> np.ndarray:
        """計算波印廷向量"""
        # S = (E × B) / μ₀
        return np.cross(electric_field, magnetic_field) / constants.mu_0
    
    def intensity(
        self,
        poynting_vector: np.ndarray
    ) -> float:
        """計算平均強度（對時間平均）"""
        return np.linalg.norm(poynting_vector) / 2
```

### 4.3 电磁波譜

| 類型 | 波長範圍 | 頻率範圍 |
|------|---------|---------|
| 無線電波 | > 1 m | < 300 MHz |
| 微波 | 1 mm - 1 m | 300 MHz - 300 GHz |
| 紅外線 | 700 nm - 1 mm | 300 GHz - 430 THz |
| 可見光 | 400 - 700 nm | 430 - 750 THz |
| 紫外線 | 10 - 400 nm | 750 THz - 30 PHz |
| X射線 | 0.01 - 10 nm | 30 PHz - 30 EHz |
| γ射線 | < 0.01 nm | > 30 EHz |

```python
class ElectromagneticSpectrum:
    """
    电磁波譜
    """
    
    SPECTRUM = {
        'radio': {'wavelength': (1, float('inf')), 'frequency': (0, 3e8)},
        'microwave': {'wavelength': (1e-3, 1), 'frequency': (3e8, 3e11)},
        'infrared': {'wavelength': (7e-7, 1e-3), 'frequency': (3e11, 4.3e14)},
        'visible': {'wavelength': (4e-7, 7e-7), 'frequency': (4.3e14, 7.5e14)},
        'ultraviolet': {'wavelength': (1e-8, 4e-7), 'frequency': (7.5e14, 3e16)},
        'xray': {'wavelength': (1e-11, 1e-8), 'frequency': (3e16, 3e19)},
        'gamma': {'wavelength': (0, 1e-11), 'frequency': (3e19, float('inf'))}
    }
    
    def classify_wavelength(self, wavelength: float) -> str:
        """根據波長分類电磁波"""
        for name, range_dict in self.SPECTRUM.items():
            w_min, w_max = range_dict['wavelength']
            if w_min <= wavelength < w_max:
                return name
        return 'unknown'
```

---

## 5. 电磁辐射 (Electromagnetic Radiation)

### 5.1 加速電荷的辐射

李納-維謝爾勢（延遲勢）：

$$\Phi(\vec{r}, t) = \frac{1}{4\pi\varepsilon_0} \left[ \frac{q}{|\vec{r} - \vec{r}_s| - \frac{\hat{n} \cdot \vec{v}}{c}} \right]_{ret}$$

$$\vec{A}(\vec{r}, t) = \frac{\mu_0}{4\pi} \left[ \frac{q\vec{v}}{|\vec{r} - \vec{r}_s| - \frac{\hat{n} \cdot \vec{v}}{c}} \right]_{ret}$$

```python
class LiénardWiechertPotentials:
    """
    李納-維謝爾勢
    """
    
    def retarded_time(
        self,
        source_position: np.ndarray,
        observation_position: np.ndarray,
        current_time: float
    ) -> float:
        """計算延遲時間"""
        # t_ret = t - |r - r_s(t_ret)| / c
        distance = np.linalg.norm(observation_position - source_position)
        return current_time - distance / constants.c
    
    def potential_electric(
        self,
        charge: float,
        source_trajectory: Callable,
        observation_position: np.ndarray,
        time: float
    ) -> float:
        """計算標量勢"""
        pass
```

### 5.2 偶極辐射

電偶極辐射的功率：

$$P = \frac{\mu_0 p_0^2 \omega^4}{12\pi c}$$

```python
class DipoleRadiation:
    """
    偶極辐射
    """
    
    def dipole_power(
        self,
        dipole_moment: float,
        frequency: float
    ) -> float:
        """計算電偶極辐射功率"""
        # P = (μ₀p₀²ω⁴)/(12πc)
        p0 = dipole_moment
        omega = 2 * np.pi * frequency
        return (constants.mu_0 * p0**2 * omega**4) / (12 * np.pi * constants.c)
    
    def radiation_pattern(
        self,
        theta: float
    ) -> float:
        """計算辐射圖案"""
        # I ∝ sin²θ
        return np.sin(theta)**2
```

### 5.3 輻射壓力

电磁波對物體的壓力：

$$P = \frac{I}{c}(1 + R)$$

其中 $R$ 為反射率。

```python
class RadiationPressure:
    """
    輻射壓力
    """
    
    def pressure(
        self,
        intensity: float,
        reflectivity: float
    ) -> float:
        """計算輻射壓力"""
        # P = I/c (1 + R)
        return intensity / constants.c * (1 + reflectivity)
    
    def solar_pressure(
        self,
        distance_au: float
    ) -> float:
        """計算太陽輻射壓力"""
        # 太陽常數在 1 AU ≈ 1361 W/m²
        solar_constant = 1361  # W/m² at 1 AU
        return solar_constant / constants.c * (1 + 0)  # 完全吸收
```

---

## 6. 电磁场与物质的交互

### 6.1 物質的电磁性質

**電極化**：

$$\vec{P} = \chi_e \varepsilon_0 \vec{E}$$

**磁化**：

$$\vec{M} = \chi_m \vec{H}$$

```python
class MaterialElectromagneticProperties:
    """
    物質的电磁性質
    """
    
    def electric_susceptibility(
        self,
        permittivity: float,
        vacuum_permittivity: float
    ) -> float:
        """計算電感受性"""
        # χ_e = ε_r - 1
        return permittivity / vacuum_permittivity - 1
    
    def magnetic_susceptibility(
        self,
        permeability: float,
        vacuum_permeability: float
    ) -> float:
        """計算磁感受性"""
        # χ_m = μ_r - 1
        return permeability / vacuum_permeability - 1
    
    def refractive_index(
        self,
        permittivity: float,
        permeability: float
    ) -> float:
        """計算折射率"""
        # n = √(ε_r μ_r)
        return np.sqrt(permittivity * permeability)
```

### 6.2 馬克斯威爾方程組（介質中）

在線性均勻介質中：

$$\nabla \cdot \vec{D} = \rho_f$$

$$\nabla \cdot \vec{B} = 0$$

$$\nabla \times \vec{E} = -\frac{\partial \vec{B}}{\partial t}$$

$$\nabla \times \vec{H} = \vec{J}_f + \frac{\partial \vec{D}}{\partial t}$$

其中：
- $\vec{D} = \varepsilon \vec{E}$ （電位移）
- $\vec{B} = \mu \vec{H}$ （磁場強度）

```python
class MaxwellEquationsMedium:
    """
    介質中的馬克斯威爾方程組
    """
    
    def wave_speed_medium(
        self,
        permittivity: float,
        permeability: float
    ) -> float:
        """介質中的光速"""
        # v = 1/√(εμ)
        return 1 / np.sqrt(permittivity * permeability)
```

### 6.3 电磁波在介質中的傳播

**折射定律**：

$$n_1 \sin\theta_1 = n_2 \sin\theta_2$$

**全內反射臨界角**：

$$\theta_c = \sin^{-1}\left(\frac{n_2}{n_1}\right)$$

```python
class WavePropagationMedium:
    """
    电磁波在介質中的傳播
    """
    
    def snell_law(
        self,
        n1: float,
        n2: float,
        theta1: float
    ) -> float:
        """斯涅爾定律"""
        # n1 sin(θ1) = n2 sin(θ2)
        return np.arcsin(n1 * np.sin(theta1) / n2)
    
    def critical_angle(
        self,
        n1: float,
        n2: float
    ) -> float:
        """全內反射臨界角"""
        if n1 <= n2:
            return float('nan')  # 不發生全內反射
        return np.arcsin(n2 / n1)
    
    def fresnel_coefficients(
        self,
        n1: float,
        n2: float,
        theta1: float
    ) -> tuple:
        """菲涅爾係數"""
        n1_over_n2 = n1 / n2
        sin_theta1 = np.sin(theta1)
        cos_theta1 = np.cos(theta1)
        
        # 判斷是否發生全內反射
        sin_theta2 = n1_over_n2 * sin_theta1
        if sin_theta2 > 1:
            return (complex(1), complex(1))  # 全反射
        
        cos_theta2 = np.sqrt(1 - sin_theta2**2)
        
        # s 偏振
        rs = (n1 * cos_theta1 - n2 * cos_theta2) / (n1 * cos_theta1 + n2 * cos_theta2)
        ts = 2 * n1 * cos_theta1 / (n1 * cos_theta1 + n2 * cos_theta2)
        
        # p 偏振
        rp = (n2 * cos_theta1 - n1 * cos_theta2) / (n2 * cos_theta1 + n1 * cos_theta2)
        tp = 2 * n1 * cos_theta1 / (n2 * cos_theta1 + n1 * cos_theta2)
        
        return (rs, ts, rp, tp)
```

---

## 7. 與其他尺度的接口

### 7.1 與古典力學 (PS-L2) 的接口

```
經典電磁學 → 古典力學：
|- 洛倫茲力：F = q(E + v × B)
|- 电磁力做功：W = ∫ F · dr
|- 电磁場的動量：p = ε₀E × B
```

### 7.2 與量子力學 (PS-L0) 的接口

```
經典電磁學 → 量子力學：
|- 電磁場的量子化：光子（自旋-1粒子）
|- 電荷的量子化：e = 1.602 × 10⁻¹⁹ C
|- 原子物理：電子殼層躍遷產生光子
|- 雷射原理：受激輻射
```

### 7.3 與狹義相對論 (PS-L3) 的接口

```
經典電磁學 → 狹義相對論：
|- 麥克斯韋方程組在洛倫茲變換下協變
|- 電場和磁場是同一四維張量的分量
|- F^{μν} = (E/c, B)
```

```python
class ElectromagnetismRelativity:
    """
    电磁學與相對論的接口
    """
    
    def electromagnetic_tensor(
        self,
        electric_field: np.ndarray,
        magnetic_field: np.ndarray
    ) -> np.ndarray:
        """构造电磁场张量 F^{μν}"""
        # F^{μν} = [0, -E/c, E/c, 0; -E/c, 0, -B, B; ...]
        c = constants.c
        F = np.zeros((4, 4))
        F[0, 1] = -electric_field[0] / c
        F[0, 2] = -electric_field[1] / c
        F[0, 3] = -electric_field[2] / c
        F[1, 0] = electric_field[0] / c
        F[2, 0] = electric_field[1] / c
        F[3, 0] = electric_field[2] / c
        
        # 磁場分量
        F[1, 2] = -magnetic_field[2]
        F[1, 3] = magnetic_field[1]
        F[2, 1] = magnetic_field[2]
        F[2, 3] = -magnetic_field[0]
        F[3, 1] = -magnetic_field[1]
        F[3, 2] = magnetic_field[0]
        
        return F
```

---

## 8. 重要常數

| 常數 | 符號 | 數值 | 單位 |
|------|------|------|------|
| 真空介電常數 | $\varepsilon_0$ | $8.854 \times 10^{-12}$ | F/m |
| 真空磁導率 | $\mu_0$ | $4\pi \times 10^{-7}$ | H/m |
| 光速 | $c$ | $2.998 \times 10^8$ | m/s |
| 基本電荷 | $e$ | $1.602 \times 10^{-19}$ | C |
| 電子質量 | $m_e$ | $9.109 \times 10^{-31}$ | kg |
| 普朗克常數 | $h$ | $6.626 \times 10^{-34}$ | J·s |

---

## 版本歷史

| 版本 | 日期 | 變更 |
|------|------|------|
| v1.0 | 2026-03-18 | 初始版本：經典電磁學完整模組 |

---

*本文檔處理經典電磁學尺度的物理框架。*
*經典電磁學是經過充分驗證的理論，適用於 PS-L2 尺度的物理現象。*
*與量子力學和相對論的和諧統一是物理學最美麗的成就之一。*
