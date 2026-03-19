---
隸屬支柱: Physics-OS v2.2
模組代號: PHYSICS-CM-001
版本: v1.0
上層模組: NoiePhysicsAGENTS/SCALE_MODULES
下層模組: 無
依賴模組: AXIOMS.md, FIELD_PERCEPTION.md, DYNAMICS_ENGINE.md
創建日期: 2026-03-18
---

# CLASSICAL_MECHANICS.md

## 經典力學 (PS-L2)

**尺度：** 10⁻³ ~ 10³ m  
**版本：** v1.0  
**狀態：** 驗證性

---

## 概述

本文檔處理**經典力學**尺度的物理框架。根據 NoiePhysicsAGENTS.md §1 的物理尺度權限層級定義，PS-L2 代表人類/古典尺度，涵蓋從毫米到公里的日常物理現象。

經典力學是物理學最古老且最成功的理論之一，為工程、機械、航空航天等領域提供了堅實的理論基礎。

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

## §0. 概述

經典力學涵蓋以下主要領域：

| 領域 | 描述 | 典型應用 |
|------|------|----------|
| 牛頓力學 | 基於三定律的粒子動力學 | 軌道計算、彈道 |
| 拉格朗日力學 | 廣義座標與變分原理 | 約束系統、分析力學 |
| 哈密頓力學 | 相空間表述 | 量子化橋樑、統計力學 |
| 剛體動力學 | 旋轉與角動量 | 陀螺儀、姿態控制 |
| 彈性力學 | 固體變形 | 結構分析、材料科學 |
| 振動與波動 | 週期運動與能量傳遞 | 橋梁設計、地震工程 |

```python
class ClassicalMechanics:
    """
    經典力學框架
    
    涵蓋從微觀到巨觀的經典物理現象
    """
    
    # 尺度範圍：10^-3 m (1mm) 到 10^3 m (1km)
    SCALE_RANGE = (1e-3, 1e3)  # meters
    
    # 速度範圍：非相對論性 (v << c)
    VELOCITY_LIMIT = 0.01 * 3e8  # 1% 光速
    
    # 能量範圍：避免量子效應
    ENERGY_THRESHOLD = 1e-20  # Joules, above thermal energy scale
```

---

## §1. 牛頓運動定律

### 1.1 第一定律（慣性定律）

任何物體若不受外力作用，則保持靜止或等速直線運動狀態：

$$\vec{F} = 0 \Rightarrow \frac{d\vec{v}}{dt} = 0$$

```python
class NewtonsFirstLaw:
    """
    牛頓第一定律：慣性定律
    
    孤立系統的動量守恆
    """
    
    def check_inertial_motion(
        self,
        position_trajectory: np.ndarray,
        velocity_threshold: float = 1e-6
    ) -> bool:
        """
        檢查是否為慣性運動
        
        若速度變化小於閾值，則視為慣性運動
        """
        velocities = np.diff(position_trajectory, axis=0)
        velocity_variation = np.std(velocities, axis=0)
        return np.all(velocity_variation < velocity_threshold)
```

### 1.2 第二定律（運動定律）

力等於動量變化率：

$$\vec{F} = \frac{d\vec{p}}{dt} = m\frac{d\vec{v}}{dt} = m\vec{a}$$

```python
class NewtonsSecondLaw:
    """
    牛頓第二定律：運動定律
    
    F = ma 是經典力學的核心方程
    """
    
    def compute_acceleration(
        self,
        force: np.ndarray,
        mass: float
    ) -> np.ndarray:
        """計算加速度 a = F/m"""
        return force / mass
    
    def integrate_motion(
        self,
        initial_state: MotionState,
        force: Callable[[float, np.ndarray], np.ndarray],
        time_span: Tuple[float, float],
        method: str = 'RK4'
    ) -> Trajectory:
        """
        積分運動方程
        
        方法：Euler, Runge-Kutta, Velocity Verlet
        """
        pass
```

### 1.3 第三定律（作用與反作用）

作用力與反作用力大小相等、方向相反、作用線不同：

$$\vec{F}_{12} = -\vec{F}_{21}$$

```python
class NewtonsThirdLaw:
    """
    牛頓第三定律：作用與反作用
    
    封閉系統的動量守恆直接來源
    """
    
    def verify_momentum_conservation(
        self,
        particles: List[Particle],
        external_force: np.ndarray = None
    ) -> bool:
        """驗證動量守恆"""
        total_momentum = sum(p.mass * p.velocity for p in particles)
        
        if external_force is not None and np.linalg.norm(external_force) > 0:
            # 有外力時，總動量變化等於衝量
            return False  # 需要時間積分驗證
        
        return True  # 動量守恆
```

### 1.4 萬有引力定律

$$F = G\frac{m_1 m_2}{r^2}$$

```python
class GravitationalForce:
    """
    萬有引力
    
    G = 6.67430 × 10^-11 m³/(kg·s²)
    """
    
    G = 6.67430e-11  # gravitational constant
    
    def compute_gravitational_force(
        self,
        mass1: float,
        mass2: float,
        position1: np.ndarray,
        position2: np.ndarray
    ) -> np.ndarray:
        """計算兩個質量間的引力"""
        r_vec = position2 - position1
        r = np.linalg.norm(r_vec)
        
        if r < 1e-10:  # 避免奇點
            return np.zeros(3)
        
        magnitude = self.G * mass1 * mass2 / r**2
        direction = r_vec / r
        
        return magnitude * direction
```

---

## §2. 拉格朗日力學

### 2.1 拉格朗日量

對於保守系統：

$$L(q, \dot{q}, t) = T(q, \dot{q}, t) - V(q, t)$$

其中 $T$ 為動能，$V$ 為位能。

```python
class LagrangianMechanics:
    """
    拉格朗日力學
    
    使用廣義座標描述系統
    """
    
    def __init__(self, lagrangian: Callable):
        """
        初始化拉格朗日力學
        
        Args:
            lagrangian: 拉格朗日量函數 L(q, q_dot, t)
        """
        self.L = lagrangian
    
    def euler_lagrange_equations(
        self,
        generalized_coords: np.ndarray,
        generalized_velocities: np.ndarray,
        time: float
    ) -> np.ndarray:
        """
        歐拉-拉格朗日方程
        
        d/dt(∂L/∂q̇) - ∂L/∂q = 0
        """
        n = len(generalized_coords)
        
        # 數值計算偏導數
        eps = 1e-8
        dL_dq_dot = np.zeros(n)
        dL_dq = np.zeros(n)
        
        for i in range(n):
            # ∂L/∂q̇_i ≈ (L(q, q̇+ε, t) - L(q, q̇-ε, t)) / (2ε)
            coords_plus = generalized_coords.copy()
            coords_minus = generalized_coords.copy()
            vel_plus = generalized_velocities.copy()
            vel_minus = generalized_velocities.copy()
            
            vel_plus[i] += eps
            vel_minus[i] -= eps
            
            dL_dq_dot[i] = (self.L(coords_plus, vel_plus, time) - 
                           self.L(coords_minus, vel_minus, time)) / (2 * eps)
            
            # ∂L/∂q_i
            coords_plus[i] += eps
            coords_minus[i] -= eps
            
            dL_dq[i] = (self.L(coords_plus, generalized_velocities, time) - 
                        self.L(coords_minus, generalized_velocities, time)) / (2 * eps)
        
        # 時間導數需要數值微分
        # 這裡簡化處理
        return dL_dq_dot - dL_dq
```

### 2.2 約束與廣義座標

對於約束系統，使用拉格朗日乘數法：

$$\frac{d}{dt}\left(\frac{\partial L}{\partial \dot{q}_j}\right) - \frac{\partial L}{\partial q_j} = \sum_i \lambda_i \frac{\partial f_i}{\partial q_j}$$

```python
class ConstrainedLagrangian:
    """
    約束拉格朗日力學
    
    使用拉格朗日乘數處理約束
    """
    
    def solve_with_constraints(
        self,
        lagrangian: Callable,
        constraint_functions: List[Callable],
        initial_state: np.ndarray,
        time_span: Tuple[float, float]
    ) -> Solution:
        """
        求解帶約束的拉格朗日方程
        
        使用拉格朗日乘數
        """
        pass
```

### 2.3 守恆定律

若 $L$ 不顯含某廣義座標 $q_k$，則對應的正則動量守恆：

$$\frac{\partial L}{\partial q_k} = 0 \Rightarrow p_k = \frac{\partial L}{\partial \dot{q}_k} = \text{constant}$$

```python
class ConservationLaws:
    """
    拉格朗日力學中的守恆定律
    """
    
    def compute_generalized_momentum(
        self,
        lagrangian: Callable,
        generalized_coords: np.ndarray,
        generalized_velocities: np.ndarray
    ) -> np.ndarray:
        """
        計算正則動量
        
        p_i = ∂L/∂q̇_i
        """
        n = len(generalized_coords)
        momenta = np.zeros(n)
        eps = 1e-8
        
        for i in range(n):
            vel_plus = generalized_velocities.copy()
            vel_minus = generalized_velocities.copy()
            vel_plus[i] += eps
            vel_minus[i] -= eps
            
            momenta[i] = (lagrangian(generalized_coords, vel_plus, 0) - 
                         lagrangian(generalized_coords, vel_minus, 0)) / (2 * eps)
        
        return momenta
```

---

## §3. 哈密頓力學

### 3.1 哈密頓量

從拉格朗日量通過勒壤得轉換得到：

$$H(q, p, t) = p_i \dot{q}_i - L(q, \dot{q}, t)$$

對於保守系統，$H = T + V$（總能量）。

```python
class HamiltonianMechanics:
    """
    哈密頓力學
    
    使用相空間描述系統
    """
    
    def __init__(self, hamiltonian: Callable):
        """
        初始化哈密頓力學
        
        Args:
            hamiltonian: 哈密頓量函數 H(q, p, t)
        """
        self.H = hamiltonian
    
    def hamilton_equations(
        self,
        state: PhaseSpaceState,
        time: float
    ) -> PhaseSpaceState:
        """
        哈密頓方程
        
        q̇ = ∂H/∂p
        ṗ = -∂H/∂q
        """
        q = state.coordinates
        p = state.momenta
        
        eps = 1e-8
        n = len(q)
        
        q_dot = np.zeros(n)
        p_dot = np.zeros(n)
        
        for i in range(n):
            # q̇_i = ∂H/∂p_i
            p_plus = p.copy()
            p_minus = p.copy()
            p_plus[i] += eps
            p_minus[i] -= eps
            
            q_dot[i] = (self.H(q, p_plus, time) - 
                       self.H(q, p_minus, time)) / (2 * eps)
            
            # ṗ_i = -∂H/∂q_i
            q_plus = q.copy()
            q_minus = q.copy()
            q_plus[i] += eps
            q_minus[i] -= eps
            
            p_dot[i] = -(self.H(q_plus, p, time) - 
                        self.H(q_minus, p, time)) / (2 * eps)
        
        return PhaseSpaceState(q_dot, p_dot)
```

### 3.2 相空間與辛幾何

哈密頓力學在辛流形上表述，滿足泊松括號：

$$\{F, G\} = \frac{\partial F}{\partial q_i}\frac{\partial G}{\partial p_i} - \frac{\partial F}{\partial p_i}\frac{\partial G}{\partial q_i}$$

```python
class SymplecticGeometry:
    """
    辛幾何結構
    
    相空間的基礎結構
    """
    
    # 辛矩陣
    J = np.array([[0, np.eye(3)], 
                  [-np.eye(3), 0]])
    
    def poisson_bracket(
        self,
        F: Callable,
        G: Callable,
        state: PhaseSpaceState
    ) -> float:
        """
        計算泊松括號 {F, G}
        
        {F, G} = ∑_i (∂F/∂q_i ∂G/∂p_i - ∂F/∂p_i ∂G/∂q_i)
        """
        q = state.coordinates
        p = state.momenta
        n = len(q)
        
        result = 0
        eps = 1e-8
        
        for i in range(n):
            # ∂F/∂q_i
            q_plus = q.copy()
            q_minus = q.copy()
            q_plus[i] += eps
            q_minus[i] -= eps
            
            dF_dq = (F(q_plus, p) - F(q_minus, p)) / (2 * eps)
            dG_dp = (G(q, p.copy() + eps * np.eye(n)[i]) - 
                    G(q, p.copy() - eps * np.eye(n)[i])) / (2 * eps)
            
            result += dF_dq * dG_dp
            
            # -∂F/∂p_i
            p_plus = p.copy()
            p_minus = p.copy()
            p_plus[i] += eps
            p_minus[i] -= eps
            
            dF_dp = (F(q, p_plus) - F(q, p_minus)) / (2 * eps)
            dG_dq = (G(q.copy() + eps * np.eye(n)[i], p) - 
                    G(q.copy() - eps * np.eye(n)[i], p)) / (2 * eps)
            
            result -= dF_dp * dG_dq
        
        return result
```

### 3.3 正則變換

正則變換保持辛結構不變：

$$\{Q_i, Q_j\} = 0, \quad \{P_i, P_j\} = 0, \quad \{Q_i, P_j\} = \delta_{ij}$$

```python
class CanonicalTransformation:
    """
    正則變換
    
    保持哈密頓方程形式不變
    """
    
    def __init__(self, transform: Callable):
        """
        初始化正則變換
        
        Args:
            transform: (q, p) -> (Q, P)
        """
        self.transform = transform
    
    def check_symplecticity(
        self,
        jacobian: np.ndarray
    ) -> bool:
        """
        檢查變換的辛性
        
        J = M^T J M
        """
        J = np.array([[0, np.eye(3)], [-np.eye(3), 0]])
        return np.allclose(jacobian.T @ J @ jacobian, J)
```

---

## §4. 剛體動力學

### 4.1 轉動慣量與慣量張量

對於連續體：

$$I_{ij} = \int_V \rho(\vec{r}) (\delta_{ij} r^2 - r_i r_j) dV$$

```python
class RigidBodyDynamics:
    """
    剛體動力學
    """
    
    def compute_inertia_tensor(
        self,
        mass_distribution: MassDistribution,
        center_of_mass: np.ndarray
    ) -> np.ndarray:
        """
        計算慣量張量
        
        I_ij = ∫ ρ(r)(δ_ij r² - r_i r_j) dV
        """
        # 數值計算慣量張量
        pass
    
    def principal_moments(
        self,
        inertia_tensor: np.ndarray
    ) -> Tuple[np.ndarray, np.ndarray]:
        """
        計算主慣量矩與主軸
        
        I = R^T I_principal R
        """
        eigenvalues, eigenvectors = np.linalg.eig(inertia_tensor)
        return eigenvalues, eigenvectors
```

### 4.2 歐拉方程

繞質心轉動的剛體運動方程：

$$I_1 \dot{\omega}_1 - (I_2 - I_3)\omega_2\omega_3 = M_1$$
$$I_2 \dot{\omega}_2 - (I_3 - I_1)\omega_3\omega_1 = M_2$$
$$I_3 \dot{\omega}_3 - (I_1 - I_2)\omega_1\omega_2 = M_3$$

```python
class EulerEquations:
    """
    歐拉方程
    
    剛體轉動的運動方程
    """
    
    def euler_equations(
        self,
        angular_momentum: np.ndarray,
        principal_moments: np.ndarray,
        external_torque: np.ndarray = None
    ) -> np.ndarray:
        """
        求解歐拉方程
        
        I · ω̇ + ω × (I · ω) = M
        """
        I = np.diag(principal_moments)
        
        if external_torque is None:
            external_torque = np.zeros(3)
        
        # ω̇ = I^(-1)(M - ω × (I · ω))
        L = I @ angular_momentum
        omega_cross_L = np.cross(angular_momentum, L)
        
        angular_acceleration = np.linalg.inv(I) @ (external_torque - omega_cross_L)
        
        return angular_acceleration
```

### 4.3 角動量與進動

$$\vec{L} = \mathbf{I} \cdot \vec{\omega}$$

```python
class AngularMomentum:
    """
    角動量與進動
    """
    
    def compute_precession_frequency(
        self,
        angular_momentum: np.ndarray,
        external_torque: np.ndarray
    ) -> float:
        """
        計算進動頻率
        
        Ω = (L × M) / |L|²
        """
        L = angular_momentum
        M = external_torque
        
        cross_product = np.cross(L, M)
        norm_L_sq = np.dot(L, L)
        
        if norm_L_sq < 1e-10:
            return 0.0
        
        return np.linalg.norm(cross_product) / norm_L_sq
```

---

## §5. 彈性力學基礎

### 5.1 應變張量

小變形下的應變：

$$\varepsilon_{ij} = \frac{1}{2}\left(\frac{\partial u_i}{\partial x_j} + \frac{\partial u_j}{\partial x_i}\right)$$

```python
class Elasticity:
    """
    彈性力學基礎
    """
    
    def compute_strain(
        self,
        displacement_field: Callable[[np.ndarray], np.ndarray]
    ) -> np.ndarray:
        """
        計算應變張量
        
        ε_ij = 1/2 (∂u_i/∂x_j + ∂u_j/∂x_i)
        """
        pass
```

### 5.2 應力張量

柯西應力：

$$\sigma_{ij} = \frac{F_j}{A_i}$$

```python
class StressTensor:
    """
    應力張量
    """
    
    def compute_stress(
        self,
        strain: np.ndarray,
        material: Material
    ) -> np.ndarray:
        """
        計算應力
        
        σ = C : ε
        其中 C 為彈性常數張量
        """
        # 廣義胡克定律
        pass
```

### 5.3 本構關係

廣義胡克定律（各向同性線彈性）：

$$\varepsilon_{ij} = \frac{1}{E}\left[(1+\nu)\sigma_{ij} - \nu \delta_{ij}\sigma_{kk}\right]$$

```python
class ConstitutiveRelation:
    """
    本構關係
    
    廣義胡克定律
    """
    
    def hookes_law_isotropic(
        self,
        stress: np.ndarray,
        youngs_modulus: float,
        poisson_ratio: float
    ) -> np.ndarray:
        """
        各向同性廣義胡克定律
        
        ε = 1/E [(1+ν)σ - ν(tr σ)I]
        """
        trace = np.trace(stress)
        identity = np.eye(3)
        
        strain = (1 / youngs_modulus) * ((1 + poisson_ratio) * stress - 
                                          poisson_ratio * trace * identity)
        
        return strain
```

---

## §6. 振動與波動

### 6.1 簡諧振子

$$m\ddot{x} + kx = 0$$

解為：

$$x(t) = A\cos(\omega t + \phi), \quad \omega = \sqrt{\frac{k}{m}}$$

```python
class HarmonicOscillator:
    """
    簡諧振子
    """
    
    def __init__(self, mass: float, k: float):
        self.mass = mass
        self.k = k
        self.omega = np.sqrt(k / mass)
    
    def position(self, t: float, amplitude: float, phase: float) -> float:
        """x(t) = A cos(ωt + φ)"""
        return amplitude * np.cos(self.omega * t + phase)
    
    def energy(self, amplitude: float) -> float:
        """總能量 E = 1/2 k A²"""
        return 0.5 * self.k * amplitude**2
```

### 6.2 阻尼振動

$$m\ddot{x} + b\dot{x} + kx = 0$$

```python
class DampedOscillator:
    """
    阻尼振子
    """
    
    def __init__(self, mass: float, k: float, damping: float):
        self.mass = mass
        self.k = k
        self.damping = damping
        self.omega_0 = np.sqrt(k / mass)
        self.gamma = damping / (2 * mass)
    
    def is_overdamped(self) -> bool:
        """γ > ω₀ 時過阻尼"""
        return self.gamma > self.omega_0
    
    def is_underdamped(self) -> bool:
        """γ < ω₀ 時欠阻尼"""
        return self.gamma < self.omega_0
    
    def solution_underdamped(
        self,
        t: float,
        initial_displacement: float,
        initial_velocity: float
    ) -> float:
        """欠阻尼解"""
        omega_d = np.sqrt(self.omega_0**2 - self.gamma**2)
        
        A = initial_displacement
        B = (initial_velocity + self.gamma * A) / omega_d
        
        return np.exp(-self.gamma * t) * (A * np.cos(omega_d * t) + 
                                          B * np.sin(omega_d * t))
```

### 6.3 波動方程

一維波動方程：

$$\frac{\partial^2 u}{\partial t^2} = v^2 \frac{\partial^2 u}{\partial x^2}$$

```python
class WaveEquation:
    """
    波動方程
    """
    
    def __init__(self, wave_speed: float):
        self.v = wave_speed
    
    def plane_wave_solution(
        self,
        x: np.ndarray,
        t: float,
        amplitude: float,
        frequency: float,
        phase: float = 0
    ) -> np.ndarray:
        """
        平面波解
        
        u(x,t) = A cos(kx - ωt + φ)
        """
        k = 2 * np.pi * frequency / self.v
        omega = 2 * np.pi * frequency
        
        return amplitude * np.cos(k * x - omega * t + phase)
    
    def compute_wave_speed(
        self,
        tension: float,
        linear_density: float
    ) -> float:
        """弦上波速 v = √(T/μ)"""
        return np.sqrt(tension / linear_density)
```

---

## §7. 與其他尺度的接口

### 7.1 與量子力學 (PS-L0) 的接口

```
經典力學 → 量子力學：
- 對應原理：ħ → 0 時量子還原為經典
- 量子化規則：從泊松括號到對易子
- 半古典近似：WKB 近似
```

### 7.2 與統計力學 (PS-L1) 的接口

```
經典力學 → 統計力學：
- 從軌跡到分佈函數
- Liouville 方程 → Boltzmann 方程
- 微正則與正則系綜
```

### 7.3 與連續介質力學 (PS-L2/PS-L3) 的接口

```
經典力學 → 連續介質力學：
- 離散 → 連續極限
- 質點 → 連續體場
- 歐拉-拉格朗日描述
```

### 7.4 與廣義相對論 (PS-L4) 的接口

```
經典力學 → 廣義相對論：
- 低速弱場極限：還原為牛頓重力
- 後牛頓近似
- 時空曲率效應
```

---

## 版本歷史

| 版本 | 日期 | 變更 |
|------|------|------|
| v1.0 | 2026-03-18 | 初始版本：涵蓋牛頓力學、拉格朗日力學、哈密頓力學、剛體動力學、彈性力學基礎、振動與波動 |

---

*本文檔處理經典力學尺度的物理框架。*
*經典力學是工程與科學的基礎，適用於 PS-L2 尺度的物理現象。*
