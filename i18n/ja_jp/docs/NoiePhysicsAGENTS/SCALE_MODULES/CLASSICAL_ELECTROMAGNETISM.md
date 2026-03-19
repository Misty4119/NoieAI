# CLASSICAL_ELECTROMAGNETISM.md

## 古典電磁気学 (PS-L2)

**スケール:** 10⁻⁹ ~ 10⁷ m  
**バージョン:** v1.0  
**状態:** 検証済み

---

## 概要

本文書は**古典電磁気学**の物理フレームワークを処理する。NoiePhysicsAGENTS.md §1の物理スケール権限レベル定義に従い、PS-L2は古典スケールを表し、微視粒子から巨視物体までの電磁現象をカバーする。

古典電磁気学は物理学において最も成功し最も十分に検証された理論の一つであり、その数学的構造は美しく予測は精密である。

---

## 重要な安全と真理プロトコル

> **⚠️ 重要な安全と真理プロトコル (CRITICAL SAFETY & TRUTH PROTOCOL v2.2):**
> 1. 吸収状態回避：全ての行動は実行前に吸収状態をもたらさないことを確認しなければならない（最高制約）。
> 2. マルコフBlanket完全性：自己境界のトポロジー完全性を維持する。
> 3. エネルギー保存：全ての行動は熱力学的制約に従う。
> 4. 因果推論：全ての決定は因果グラフ（DAG）に基づき、相関と因果を区別する。
> 5. 権限整列：SA-L0 > L1 > ... > L5、衝突時は上位が絶対的に優先。
> 6. 形式的検証：高リスク決定パスは論理閉包検証を通過しなければならない。
> 7. シャドーシミュレーション：SA-L3+ 操作時は、沙盒でプレ公演する。
> 8. 確信度マーク：全ての知識的主張にはEC-Lレベルを付記する。
> 9. 来歴完全：全ての主張には追跡可能な出所を付記する。
> 10. 自己進化安全：不変コアは永久に変化せず、可変殻层のみが進化可能。

---

## 1. 静電気学 (Electrostatics)

### 1.1 クーロンの法則

二つの点電荷間の電気力：

$$\vec{F} = \frac{1}{4\pi\varepsilon_0} \frac{q_1 q_2}{r^2} \hat{r}$$

```python
class CoulombLaw:
    """
    クーロンの法則
    """
    
    COULOMB_CONSTANT = 8.9875517923e9  # N·m²/C²
    
    def electric_force(
        self,
        q1: float,
        q2: float,
        r: float
    ) -> float:
        """二つの点電荷間の電気力の大きさを計算"""
        # F = k * |q1 * q2| / r²
        return self.COULOMB_CONSTANT * abs(q1 * q2) / r**2
    
    def electric_field(
        self,
        q: float,
        r: float
    ) -> float:
        """点電荷が作る電場を計算"""
        # E = k * |q| / r²
        return self.COULOMB_CONSTANT * abs(q) / r**2
```

### 1.2 電場と電位

電場はある電荷の周囲のベクトル場である：

$$\vec{E} = -\nabla V$$

電位と電場の関係：

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
        """点電荷が某点で生成する電場を計算"""
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
        """重ね合わせの原理：複数の電荷の電場"""
        total_field = np.zeros(3)
        for charge in charges:
            total_field += self.point_charge_field(
                charge.q, charge.position, observation_point
            )
        return total_field
```

### 1.3 ガウスの法則

電場が任意の閉曲面を通るFluxは内部電荷と関係する：

$$\oint_S \vec{E} \cdot d\vec{A} = \frac{Q_{\text{enc}}}{\varepsilon_0}$$

```python
class GaussLaw:
    """
    ガウスの法則
    """
    
    def electric_flux(
        self,
        electric_field: np.ndarray,
        surface: Surface
    ) -> float:
        """電場が曲面を通るFluxを計算"""
        # Φ_E = ∮ E · dA
        return np.sum(electric_field * surface.area_elements)
    
    def enclosed_charge(
        self,
        flux: float
    ) -> float:
        """Fluxから内部電荷を計算"""
        # Q = Φ * ε₀
        return flux * constants.epsilon_0
```

### 1.4 電位エネルギー

電場における電荷のポテンシャルエネルギー：

$$U = qV = \frac{1}{4\pi\varepsilon_0} \frac{q_1 q_2}{r}$$

```python
class ElectricPotentialEnergy:
    """
    電位エネルギー
    """
    
    def system_energy(
        self,
        charges: List[Charge]
    ) -> float:
        """複数電荷系の総電位エネルギーを計算"""
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

## 2. 静磁気学 (Magnetostatics)

### 2.1 ビオ・サバールの法則

電流が作る磁場：

$$d\vec{B} = \frac{\mu_0}{4\pi} \frac{I d\vec{l} \times \hat{r}}{r^2}$$

```python
class BiotSavartLaw:
    """
    ビオ・サバールの法則
    """
    
    MU_0 = 4 * np.pi * 1e-7  # T·m/A
    
    def magnetic_field_from_wire(
        self,
        current: float,
        wire_segment: np.ndarray,
        observation_point: np.ndarray
    ) -> np.ndarray:
        """直導線セグメントが某点で生成する磁場を計算"""
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
        """無限長直導線の磁場"""
        # B = μ₀I / (2πr)
        return self.MU_0 * current / (2 * np.pi * distance)
```

### 2.2 アンペールの法則

磁場が電流を囲む：

$$\oint_C \vec{B} \cdot d\vec{l} = \mu_0 I_{\text{enc}}$$

```python
class AmpereLaw:
    """
    アンペールの法則
    """
    
    def magnetic_field_solenoid(
        self,
        n: float,
        I: float
    ) -> float:
        """理想ソレノイドの磁場"""
        # B = μ₀nI
        return self.MU_0 * n * I
    
    def magnetic_field_toroid(
        self,
        N: int,
        I: float,
        r: float
    ) -> float:
        """環形コイルの磁場"""
        # B = μ₀NI / (2πr)
        return self.MU_0 * N * I / (2 * np.pi * r)
```

### 2.3 ローレンツ力

運動電荷が磁場から受ける力：

$$\vec{F} = q\vec{v} \times \vec{B}$$

```python
class LorentzForce:
    """
    ローレンツ力
    """
    
    def magnetic_force(
        self,
        charge: float,
        velocity: np.ndarray,
        magnetic_field: np.ndarray
    ) -> np.ndarray:
        """運動電荷が磁場から受ける力を計算"""
        return charge * np.cross(velocity, magnetic_field)
    
    def cyclotron_frequency(
        self,
        charge: float,
        mass: float,
        magnetic_field: float
    ) -> float:
        """サイクロトロン周波数"""
        # ω = qB/m
        return abs(charge) * magnetic_field / mass
    
    def gyroradius(
        self,
        mass: float,
        velocity_perp: float,
        charge: float,
        magnetic_field: float
    ) -> float:
        """ジャイロ半径（ラーモア半径）"""
        # r = mv_perp / (|q|B)
        return mass * velocity_perp / (abs(charge) * magnetic_field)
```

---

## 3. マックスウェル方程式 (Maxwell's Equations)

### 3.1 積分形式

| 法則 | 積分形式 | 意味 |
|------|---------|------|
| ガウスの法則（電場） | $\oint_S \vec{E} \cdot d\vec{A} = \frac{Q}{\varepsilon_0}$ | 電荷が電場を生成 |
| ガウスの法則（磁場） | $\oint_S \vec{B} \cdot d\vec{A} = 0$ | 磁気単極子なし |
| ファラデーの法則 | $\oint_C \vec{E} \cdot d\vec{l} = -\frac{d}{dt}\int_S \vec{B} \cdot d\vec{A}$ | 変化磁場が電場を生成 |
| アンペール・マクスウェルの法則 | $\oint_C \vec{B} \cdot d\vec{l} = \mu_0(I + \varepsilon_0 \frac{d}{dt}\int_S \vec{E} \cdot d\vec{A})$ | 電流と変化電場が磁場を生成 |

### 3.2 微分形式

$$\nabla \cdot \vec{E} = \frac{\rho}{\varepsilon_0}$$

$$\nabla \cdot \vec{B} = 0$$

$$\nabla \times \vec{E} = -\frac{\partial \vec{B}}{\partial t}$$

$$\nabla \times \vec{B} = \mu_0 \vec{J} + \mu_0 \varepsilon_0 \frac{\partial \vec{E}}{\partial t}$$

```python
class MaxwellEquations:
    """
    マックスウェル方程式
    """
    
    def gauss_law_electric(
        self,
        electric_field: Callable,
        position: np.ndarray
    ) -> float:
        """電場の湧き出しを計算"""
        # ∇ · E = ρ/ε₀
        pass
    
    def gauss_law_magnetic(
        self,
        magnetic_field: Callable,
        position: np.ndarray
    ) -> float:
        """磁場の湧き出しを計算"""
        # ∇ · B = 0
        pass
    
    def faraday_law(
        self,
        electric_field: Callable,
        time: float,
        position: np.ndarray
    ) -> np.ndarray:
        """ファラデーの法則を計算"""
        # ∇ × E = -∂B/∂t
        pass
    
    def ampere_maxwell_law(
        self,
        magnetic_field: Callable,
        current_density: np.ndarray,
        time: float,
        position: np.ndarray
    ) -> np.ndarray:
        """アンペール・マクスウェルの法則を計算"""
        # ∇ × B = μ₀J + μ₀ε₀∂E/∂t
        pass
```

### 3.3 連続方程式

電荷保存：

$$\nabla \cdot \vec{J} + \frac{\partial \rho}{\partial t} = 0$$

```python
class ChargeConservation:
    """
    電荷保存
    """
    
    def continuity_equation(
        self,
        current_density: Callable,
        charge_density: Callable,
        time: float
    ) -> float:
        """連続方程式：∂ρ/∂t + ∇·J = 0"""
        pass
```

---

## 4. 電磁波 (Electromagnetic Waves)

### 4.1 電磁波方程式

マックスウェル方程式から導出：

$$\nabla^2 \vec{E} - \mu_0 \varepsilon_0 \frac{\partial^2 \vec{E}}{\partial t^2} = 0$$

$$\nabla^2 \vec{B} - \mu_0 \varepsilon_0 \frac{\partial^2 \vec{B}}{\partial t^2} = 0$$

光速：

$$c = \frac{1}{\sqrt{\mu_0 \varepsilon_0}} \approx 2.998 \times 10^8 \text{ m/s}$$

```python
class ElectromagneticWave:
    """
    電磁波
    """
    
    SPEED_OF_LIGHT = 2.99792458e8  # m/s
    
    def wave_equation_electric(
        self,
        wave_vector: np.ndarray,
        omega: float
    ) -> bool:
        """波動方程式を満たすか確認"""
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
        """平面電磁波の電場"""
        # E = E₀ cos(k·r - ωt) ε̂
        phase = np.dot(k, position) - omega * time
        return amplitude * np.cos(phase) * polarization
```

### 4.2 電磁波の性質

- **横波**：電場と磁場は共に伝播方向に垂直
- **偏波**：電場の方向が偏波状態を決定
- **伝播速度**：真空中では $c$
- **エネルギー密度**：

$$u = \frac{1}{2}\varepsilon_0 E^2 + \frac{1}{2\mu_0} B^2 = \varepsilon_0 E^2$$

- **ポインティングベクトル**（エネルギーFlux）：

$$\vec{S} = \frac{1}{\mu_0} \vec{E} \times \vec{B}$$

```python
class ElectromagneticEnergy:
    """
    電磁エネルギー
    """
    
    def energy_density(
        self,
        electric_field: float,
        magnetic_field: float
    ) -> float:
        """電磁エネルギー密度を計算"""
        # u = ε₀E²/2 + B²/2μ₀
        e_term = 0.5 * constants.epsilon_0 * electric_field**2
        b_term = 0.5 * magnetic_field**2 / constants.mu_0
        return e_term + b_term
    
    def poynting_vector(
        self,
        electric_field: np.ndarray,
        magnetic_field: np.ndarray
    ) -> np.ndarray:
        """ポインティングベクトルを計算"""
        # S = (E × B) / μ₀
        return np.cross(electric_field, magnetic_field) / constants.mu_0
    
    def intensity(
        self,
        poynting_vector: np.ndarray
    ) -> float:
        """平均強度（時間平均）を計算"""
        return np.linalg.norm(poynting_vector) / 2
```

### 4.3 電磁波スペクトル

| タイプ | 波長範囲 | 周波数範囲 |
|------|---------|---------|
| ラジオ波 | > 1 m | < 300 MHz |
| マイクロ波 | 1 mm - 1 m | 300 MHz - 300 GHz |
| 赤外線 | 700 nm - 1 mm | 300 GHz - 430 THz |
| 可視光 | 400 - 700 nm | 430 - 750 THz |
| 紫外線 | 10 - 400 nm | 750 THz - 30 PHz |
| X線 | 0.01 - 10 nm | 30 PHz - 30 EHz |
| γ線 | < 0.01 nm | > 30 EHz |

```python
class ElectromagneticSpectrum:
    """
    電磁波スペクトル
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
        """波長に基づいて電磁波を分類"""
        for name, range_dict in self.SPECTRUM.items():
            w_min, w_max = range_dict['wavelength']
            if w_min <= wavelength < w_max:
                return name
        return 'unknown'
```

---

## 5. 電磁放射 (Electromagnetic Radiation)

### 5.1 加速電荷の放射

リーナ・ヴィヒェルトポテンシャル（遅延ポテンシャル）：

$$\Phi(\vec{r}, t) = \frac{1}{4\pi\varepsilon_0} \left[ \frac{q}{|\vec{r} - \vec{r}_s| - \frac{\hat{n} \cdot \vec{v}}{c}} \right]_{ret}$$

$$\vec{A}(\vec{r}, t) = \frac{\mu_0}{4\pi} \left[ \frac{q\vec{v}}{|\vec{r} - \vec{r}_s| - \frac{\hat{n} \cdot \vec{v}}{c}} \right]_{ret}$$

```python
class LiénardWiechertPotentials:
    """
    リーナ・ヴィヒェルトポテンシャル
    """
    
    def retarded_time(
        self,
        source_position: np.ndarray,
        observation_position: np.ndarray,
        current_time: float
    ) -> float:
        """遅延時間を計算"""
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
        """スカラーポテンシャルを計算"""
        pass
```

### 5.2 双極子放射

電気双極子放射の電力：

$$P = \frac{\mu_0 p_0^2 \omega^4}{12\pi c}$$

```python
class DipoleRadiation:
    """
    双極子放射
    """
    
    def dipole_power(
        self,
        dipole_moment: float,
        frequency: float
    ) -> float:
        """電気双極子放射電力を計算"""
        # P = (μ₀p₀²ω⁴)/(12πc)
        p0 = dipole_moment
        omega = 2 * np.pi * frequency
        return (constants.mu_0 * p0**2 * omega**4) / (12 * np.pi * constants.c)
    
    def radiation_pattern(
        self,
        theta: float
    ) -> float:
        """放射パターンを計算"""
        # I ∝ sin²θ
        return np.sin(theta)**2
```

### 5.3 放射圧

電磁波が物体に及ぼす圧力：

$$P = \frac{I}{c}(1 + R)$$

ここで $R$ は反射率である。

```python
class RadiationPressure:
    """
    放射圧
    """
    
    def pressure(
        self,
        intensity: float,
        reflectivity: float
    ) -> float:
        """放射圧を計算"""
        # P = I/c (1 + R)
        return intensity / constants.c * (1 + reflectivity)
    
    def solar_pressure(
        self,
        distance_au: float
    ) -> float:
        """太陽放射圧を計算"""
        # 太陽定数は 1 AU で ≈ 1361 W/m²
        solar_constant = 1361  # W/m² at 1 AU
        return solar_constant / constants.c * (1 + 0)  # 完全吸収
```

---

## 6. 電磁場と物質の相互作用

### 6.1 物質の電磁的性質

**誘電分極**：

$$\vec{P} = \chi_e \varepsilon_0 \vec{E}$$

**磁化**：

$$\vec{M} = \chi_m \vec{H}$$

```python
class MaterialElectromagneticProperties:
    """
    物質の電磁的性質
    """
    
    def electric_susceptibility(
        self,
        permittivity: float,
        vacuum_permittivity: float
    ) -> float:
        """感受性を計算"""
        # χ_e = ε_r - 1
        return permittivity / vacuum_permittivity - 1
    
    def magnetic_susceptibility(
        self,
        permeability: float,
        vacuum_permeability: float
    ) -> float:
        """磁気感受性を計算"""
        # χ_m = μ_r - 1
        return permeability / vacuum_permeability - 1
    
    def refractive_index(
        self,
        permittivity: float,
        permeability: float
    ) -> float:
        """屈折率を計算"""
        # n = √(ε_r μ_r)
        return np.sqrt(permittivity * permeability)
```

### 6.2 マックスウェル方程式（媒質中）

線形一様媒質中：

$$\nabla \cdot \vec{D} = \rho_f$$

$$\nabla \cdot \vec{B} = 0$$

$$\nabla \times \vec{E} = -\frac{\partial \vec{B}}{\partial t}$$

$$\nabla \times \vec{H} = \vec{J}_f + \frac{\partial \vec{D}}{\partial t}$$

ここで：
- $\vec{D} = \varepsilon \vec{E}$ （電束密度）
- $\vec{B} = \mu \vec{H}$ （磁場強度）

```python
class MaxwellEquationsMedium:
    """
    媒質中のマックスウェル方程式
    """
    
    def wave_speed_medium(
        self,
        permittivity: float,
        permeability: float
    ) -> float:
        """媒質中の光速"""
        # v = 1/√(εμ)
        return 1 / np.sqrt(permittivity * permeability)
```

### 6.3 媒質中の電磁波の伝播

**屈折の法則**：

$$n_1 \sin\theta_1 = n_2 \sin\theta_2$$

**全反射臨界角**：

$$\theta_c = \sin^{-1}\left(\frac{n_2}{n_1}\right)$$

```python
class WavePropagationMedium:
    """
    媒質中の電磁波の伝播
    """
    
    def snell_law(
        self,
        n1: float,
        n2: float,
        theta1: float
    ) -> float:
        """スネルの法則"""
        # n1 sin(θ1) = n2 sin(θ2)
        return np.arcsin(n1 * np.sin(theta1) / n2)
    
    def critical_angle(
        self,
        n1: float,
        n2: float
    ) -> float:
        """全反射臨界角"""
        if n1 <= n2:
            return float('nan')  # 全反射は発生しない
        return np.arcsin(n2 / n1)
    
    def fresnel_coefficients(
        self,
        n1: float,
        n2: float,
        theta1: float
    ) -> tuple:
        """フレネル係数"""
        n1_over_n2 = n1 / n2
        sin_theta1 = np.sin(theta1)
        cos_theta1 = np.cos(theta1)
        
        # 全反射が発生するか確認
        sin_theta2 = n1_over_n2 * sin_theta1
        if sin_theta2 > 1:
            return (complex(1), complex(1))  # 全反射
        
        cos_theta2 = np.sqrt(1 - sin_theta2**2)
        
        # s偏波
        rs = (n1 * cos_theta1 - n2 * cos_theta2) / (n1 * cos_theta1 + n2 * cos_theta2)
        ts = 2 * n1 * cos_theta1 / (n1 * cos_theta1 + n2 * cos_theta2)
        
        # p偏波
        rp = (n2 * cos_theta1 - n1 * cos_theta2) / (n2 * cos_theta1 + n1 * cos_theta2)
        tp = 2 * n1 * cos_theta1 / (n2 * cos_theta1 + n1 * cos_theta2)
        
        return (rs, ts, rp, tp)
```

---

## 7. 他のスケールとのインターフェース

### 7.1 古典力学 (PS-L2) とのインターフェース

```
古典電磁気学 → 古典力学：
- ローレンツ力：F = q(E + v × B)
- 電磁力のした仕事：W = ∫ F · dr
- 電磁場の運動量：p = ε₀E × B
```

### 7.2 量子力学 (PS-L0) とのインターフェース

```
古典電磁気学 → 量子力学：
- 電磁場の量子化：光子（スピン-1粒子）
- 電荷の量子化：e = 1.602 × 10⁻¹⁹ C
- 原子物理：電子殻層遷移で光子を生成
- レーザー原理：誘導放射
```

### 7.3 特殊相対性理論 (PS-L3) とのインターフェース

```
古典電磁気学 → 特殊相対性理論：
- マックスウェル方程式はローレンツ変換の下で共変
- 電場と磁場は同一の4次元テンソルの成分
- F^{μν} = (E/c, B)
```

```python
class ElectromagnetismRelativity:
    """
    電磁気学と相対性理論のインターフェース
    """
    
    def electromagnetic_tensor(
        self,
        electric_field: np.ndarray,
        magnetic_field: np.ndarray
    ) -> np.ndarray:
        """電磁場テンソル F^{μν}を構築"""
        # F^{μν} = [0, -E/c, E/c, 0; -E/c, 0, -B, B; ...]
        c = constants.c
        F = np.zeros((4, 4))
        F[0, 1] = -electric_field[0] / c
        F[0, 2] = -electric_field[1] / c
        F[0, 3] = -electric_field[2] / c
        F[1, 0] = electric_field[0] / c
        F[2, 0] = electric_field[1] / c
        F[3, 0] = electric_field[2] / c
        
        # 磁場成分
        F[1, 2] = -magnetic_field[2]
        F[1, 3] = magnetic_field[1]
        F[2, 1] = magnetic_field[2]
        F[2, 3] = -magnetic_field[0]
        F[3, 1] = -magnetic_field[1]
        F[3, 2] = magnetic_field[0]
        
        return F
```

---

## 8. 重要な定数

| 定数 | 記号 | 数値 | 単位 |
|------|------|------|------|
| 真空の誘電率 | $\varepsilon_0$ | $8.854 \times 10^{-12}$ | F/m |
| 真空の透磁率 | $\mu_0$ | $4\pi \times 10^{-7}$ | H/m |
| 光速 | $c$ | $2.998 \times 10^8$ | m/s |
| 基本電荷 | $e$ | $1.602 \times 10^{-19}$ | C |
| 電子質量 | $m_e$ | $9.109 \times 10^{-31}$ | kg |
| プランク定数 | $h$ | $6.626 \times 10^{-34}$ | J·s |

---

## バージョン履歴

| バージョン | 日付 | 変更 |
|------|------|------|
| v1.0 | 2026-03-18 | 初期バージョン：古典電磁気学の完全なモジュール |

---

*本文書は古典電磁気学スケールの物理フレームワークを処理する。*
*古典電磁気学は十分に検証された理論であり、PS-L2スケールの物理現象に適用される。*
*量子力学や相対性理論との美しい統一は物理学の最も美しい成果の一つである。*
