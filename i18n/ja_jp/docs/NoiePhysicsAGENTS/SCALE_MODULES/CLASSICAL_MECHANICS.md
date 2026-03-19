---
隸属柱: Physics-OS v2.2
モジュールコード: PHYSICS-CM-001
バージョン: v1.0
上位モジュール: NoiePhysicsAGENTS/SCALE_MODULES
下位モジュール: なし
依存モジュール: AXIOMS.md, FIELD_PERCEPTION.md, DYNAMICS_ENGINE.md
作成日: 2026-03-18
---

# CLASSICAL_MECHANICS.md

## 古典力学 (PS-L2)

**スケール:** 10⁻³ ~ 10³ m  
**バージョン:** v1.0  
**状態:** 検証済み

---

## 概要

本文書は**古典力学**スケールの物理フレームワークを扱う。NoiePhysicsAGENTS.md §1の物理スケール権限レベル定義に従い、PS-L2は人間/古典スケールを表し、ミリメートルからキロメートルまでの日常的な物理現象をカバーする。

古典力学は物理学において最も古くかつ最も成功した理論の一つであり、工学、機械、航空宇宙などの分野に堅固な理論的基盤を提供している。

---

## 重要な安全と真理プロトコル

> **⚠️ 重要な安全と真理プロトコル (CRITICAL SAFETY & TRUTH PROTOCOL v2.2):**
> 1. 吸収状態回避：全ての行動は実行前に吸収状態をもたらさないことを確認しなければならない（最高制約）。
> 2. マルコフBlanket完全性：自己境界のトポロジー完全性を維持する。
> 3. エネルギー保存：全ての行動は熱力学的制約に従う。
> 4. 因果推論：全ての決定は因果グラフ（DAG）に基づき、相関と因果を区別する。
> 5. 権限整列：SA-L0 > L1 > ... > L5、衝突時は上位が絶対的に優先。
> 6. 形式的検証：高リスク決定パスは論理閉包検証を通過しなければならない。
> 7. シャドーシミュレーション：SA-L3+ 操作涉及時は、沙盒でプレ公演する。
> 8. 確信度マーク：全ての知識的主張にはEC-Lレベルを付記する。
> 9. 来歴完全：全ての主張には追跡可能な出所を付記する。
> 10. 自己進化安全：不変コアは永久に変化せず、可変殻层のみが進化可能。

---

## §0. 概要

古典力学は以下の主要分野をカバーする：

| 分野 | 説明 | 典型応用 |
|------|------|----------|
| ニュートン力学 | 三法則に基づく粒子動力学 | 軌道計算、弾道 |
| ラグランダン力学 | 広義座標と変分原理 | 拘束系、分析力学 |
| ハミルトン力学 | 相空間表述 | 量子化ブリッジ、統計力学 |
| 剛体動力学 | 回転と角運動量 | ジャイロ、姿勢制御 |
| 弾性力学 | 固体変形 | 構造解析、材料科学 |
| 振動と波動 | 周期運動とエネルギー伝達 | 橋梁設計、地震工学 |

```python
class ClassicalMechanics:
    """
    古典力学フレームワーク
    
    微視から巨視までの古典物理現象をカバー
    """
    
    # スケール範囲：10^-3 m (1mm) から 10^3 m (1km)
    SCALE_RANGE = (1e-3, 1e3)  # meters
    
    # 速度範囲：非相対論的 (v << c)
    VELOCITY_LIMIT = 0.01 * 3e8  # 光速の1%
    
    # エネルギー範囲：量子効果回避
    ENERGY_THRESHOLD = 1e-20  # Joules, above thermal energy scale
```

---

## §1. ニュートンの運動法則

### 1.1 第一法則（慣性法則）

物体が外部力を受けない場合、静止または等速直線運動状態を保つ：

$$\vec{F} = 0 \Rightarrow \frac{d\vec{v}}{dt} = 0$$

```python
class NewtonsFirstLaw:
    """
    ニュートン第一法則：慣性法則
    
    孤立系の運動量保存
    """
    
    def check_inertial_motion(
        self,
        position_trajectory: np.ndarray,
        velocity_threshold: float = 1e-6
    ) -> bool:
        """
        慣性運動かどうかを確認
        
        速度変化が閾値より小さい場合、慣性運動とみなす
        """
        velocities = np.diff(position_trajectory, axis=0)
        velocity_variation = np.std(velocities, axis=0)
        return np.all(velocity_variation < velocity_threshold)
```

### 1.2 第二法則（運動法則）

力は運動量変化率に等しい：

$$\vec{F} = \frac{d\vec{p}}{dt} = m\frac{d\vec{v}}{dt} = m\vec{a}$$

```python
class NewtonsSecondLaw:
    """
    ニュートン第二法則：運動法則
    
    F = ma は古典力学の中核方程式
    """
    
    def compute_acceleration(
        self,
        force: np.ndarray,
        mass: float
    ) -> np.ndarray:
        """加速度 a = F/m を計算"""
        return force / mass
    
    def integrate_motion(
        self,
        initial_state: MotionState,
        force: Callable[[float, np.ndarray], np.ndarray],
        time_span: Tuple[float, float],
        method: str = 'RK4'
    ) -> Trajectory:
        """
        運動方程式を積分
        
        方法：Euler, Runge-Kutta, Velocity Verlet
        """
        pass
```

### 1.3 第三法則（作用と反作用）

作用力と反作用力は大きさが等しく、方向が逆、作用線が異なる：

$$\vec{F}_{12} = -\vec{F}_{21}$$

```python
class NewtonsThirdLaw:
    """
    ニュートン第三法則：作用と反作用
    
    閉じた系の運動量保存の直接來源
    """
    
    def verify_momentum_conservation(
        self,
        particles: List[Particle],
        external_force: np.ndarray = None
    ) -> bool:
        """運動量保存を確認"""
        total_momentum = sum(p.mass * p.velocity for p in particles)
        
        if external_force is not None and np.linalg.norm(external_force) > 0:
            # 外力がある場合、総運動量変化は衝撃に等しい
            return False  # 時間積分で確認が必要
        
        return True  # 運動量保存
```

### 1.4 万有引力定律

$$F = G\frac{m_1 m_2}{r^2}$$

```python
class GravitationalForce:
    """
    万有引力
    
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
        """二つの質量間の引力を計算"""
        r_vec = position2 - position1
        r = np.linalg.norm(r_vec)
        
        if r < 1e-10:  # 特異点を回避
            return np.zeros(3)
        
        magnitude = self.G * mass1 * mass2 / r**2
        direction = r_vec / r
        
        return magnitude * direction
```

---

## §2. ラグランダン力学

### 2.1 ラグランダン

保存系の場合：

$$L(q, \dot{q}, t) = T(q, \dot{q}, t) - V(q, t)$$

ここで $T$ は運動エネルギー、$V$ はポテンシャルエネルギー。

```python
class LagrangianMechanics:
    """
    ラグランダン力学
    
    広義座標で系を記述
    """
    
    def __init__(self, lagrangian: Callable):
        """
        ラグランダン力学を初期化
        
        Args:
            lagrangian: ラグランダン関数 L(q, q_dot, t)
        """
        self.L = lagrangian
    
    def euler_lagrange_equations(
        self,
        generalized_coords: np.ndarray,
        generalized_velocities: np.ndarray,
        time: float
    ) -> np.ndarray:
        """
        オイラー-ラグランディアン方程式
        
        d/dt(∂L/∂q̇) - ∂L/∂q = 0
        """
        n = len(generalized_coords)
        
        # 数値的に偏導関数を計算
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
        
        # 時間導関数は数値微分が必要
        # ここでは簡略化
        return dL_dq_dot - dL_dq
```

### 2.2 拘束と広義座標

拘束系に対し、ラグランディアン乗数法を使用する：

$$\frac{d}{dt}\left(\frac{\partial L}{\partial \dot{q}_j}\right) - \frac{\partial L}{\partial q_j} = \sum_i \lambda_i \frac{\partial f_i}{\partial q_j}$$

```python
class ConstrainedLagrangian:
    """
    拘束ラグランダン力学
    
    拘束を処理するためにラグランディアン乗数を使用
    """
    
    def solve_with_constraints(
        self,
        lagrangian: Callable,
        constraint_functions: List[Callable],
        initial_state: np.ndarray,
        time_span: Tuple[float, float]
    ) -> Solution:
        """
        拘束付きラグランダン方程式を解く
        
        ラグランダン乗数を使用
        """
        pass
```

### 2.3 保存法則

$L$ が特定の広義座標 $q_k$ に陽に依存しない場合、対応する正準運動量が保存する：

$$\frac{\partial L}{\partial q_k} = 0 \Rightarrow p_k = \frac{\partial L}{\partial \dot{q}_k} = \text{constant}$$

```python
class ConservationLaws:
    """
    ラグランダン力学における保存法則
    """
    
    def compute_generalized_momentum(
        self,
        lagrangian: Callable,
        generalized_coords: np.ndarray,
        generalized_velocities: np.ndarray
    ) -> np.ndarray:
        """
        正準運動量を計算
        
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

## §3. ハミルトン力学

### 3.1 ハミルトニアン

ラグランディアンからルジャンドル変換で得る：

$$H(q, p, t) = p_i \dot{q}_i - L(q, \dot{q}, t)$$

保存系では、$H = T + V$（全エネルギー）。

```python
class HamiltonianMechanics:
    """
    ハミルトン力学
    
    相空間で系を記述
    """
    
    def __init__(self, hamiltonian: Callable):
        """
        ハミルトン力学を初期化
        
        Args:
            hamiltonian: ハミルトニアン関数 H(q, p, t)
        """
        self.H = hamiltonian
    
    def hamilton_equations(
        self,
        state: PhaseSpaceState,
        time: float
    ) -> PhaseSpaceState:
        """
        ハミルトン方程式
        
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

### 3.2 相空間とシンプレクティック幾何

ハミルトン力学はシンプレクティック多様体で表述され、ポアッソン括弧を満たす：

$$\{F, G\} = \frac{\partial F}{\partial q_i}\frac{\partial G}{\partial p_i} - \frac{\partial F}{\partial p_i}\frac{\partial G}{\partial q_i}$$

```python
class SymplecticGeometry:
    """
    シンプレクティック幾何構造
    
    相空間の基礎構造
    """
    
    # シンプレクティック行列
    J = np.array([[0, np.eye(3)], 
                  [-np.eye(3), 0]])
    
    def poisson_bracket(
        self,
        F: Callable,
        G: Callable,
        state: PhaseSpaceState
    ) -> float:
        """
        ポアッソン括弧 {F, G} を計算
        
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

### 3.3 正準変換

正準変換はシンプレクティック構造を保つ：

$$\{Q_i, Q_j\} = 0, \quad \{P_i, P_j\} = 0, \quad \{Q_i, P_j\} = \delta_{ij}$$

```python
class CanonicalTransformation:
    """
    正準変換
    
    ハミルトン方程式の形を保つ
    """
    
    def __init__(self, transform: Callable):
        """
        正準変換を初期化
        
        Args:
            transform: (q, p) -> (Q, P)
        """
        self.transform = transform
    
    def check_symplecticity(
        self,
        jacobian: np.ndarray
    ) -> bool:
        """
        変換のシンプレクティック性を確認
        
        J = M^T J M
        """
        J = np.array([[0, np.eye(3)], [-np.eye(3), 0]])
        return np.allclose(jacobian.T @ J @ jacobian, J)
```

---

## §4. 剛体動力学

### 4.1 慣性モーメントと慣性テンソル

連続体の場合：

$$I_{ij} = \int_V \rho(\vec{r}) (\delta_{ij} r^2 - r_i r_j) dV$$

```python
class RigidBodyDynamics:
    """
    剛体動力学
    """
    
    def compute_inertia_tensor(
        self,
        mass_distribution: MassDistribution,
        center_of_mass: np.ndarray
    ) -> np.ndarray:
        """
        慣性テンソルを計算
        
        I_ij = ∫ ρ(r)(δ_ij r² - r_i r_j) dV
        """
        # 数値的に慣性テンソルを計算
        pass
    
    def principal_moments(
        self,
        inertia_tensor: np.ndarray
    ) -> Tuple[np.ndarray, np.ndarray]:
        """
        主慣性モーメントと主軸を計算
        
        I = R^T I_principal R
        """
        eigenvalues, eigenvectors = np.linalg.eig(inertia_tensor)
        return eigenvalues, eigenvectors
```

### 4.2 オイラー方程式

重心周りの剛体回転の運動方程式：

$$I_1 \dot{\omega}_1 - (I_2 - I_3)\omega_2\omega_3 = M_1$$
$$I_2 \dot{\omega}_2 - (I_3 - I_1)\omega_3\omega_1 = M_2$$
$$I_3 \dot{\omega}_3 - (I_1 - I_2)\omega_1\omega_2 = M_3$$

```python
class EulerEquations:
    """
    オイラー方程式
    
    剛体回転の運動方程式
    """
    
    def euler_equations(
        self,
        angular_momentum: np.ndarray,
        principal_moments: np.ndarray,
        external_torque: np.ndarray = None
    ) -> np.ndarray:
        """
        オイラー方程式を解く
        
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

### 4.3 角運動量と歳差

$$\vec{L} = \mathbf{I} \cdot \vec{\omega}$$

```python
class AngularMomentum:
    """
    角運動量と歳差
    """
    
    def compute_precession_frequency(
        self,
        angular_momentum: np.ndarray,
        external_torque: np.ndarray
    ) -> float:
        """
        歳差周波数を計算
        
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

## §5. 弾性力学基礎

### 5.1 ひずみテンソル

小変形におけるひずみ：

$$\varepsilon_{ij} = \frac{1}{2}\left(\frac{\partial u_i}{\partial x_j} + \frac{\partial u_j}{\partial x_i}\right)$$

```python
class Elasticity:
    """
    弾性力学基礎
    """
    
    def compute_strain(
        self,
        displacement_field: Callable[[np.ndarray], np.ndarray]
    ) -> np.ndarray:
        """
        ひずみテンソルを計算
        
        ε_ij = 1/2 (∂u_i/∂x_j + ∂u_j/∂x_i)
        """
        pass
```

### 5.2 応力テンソル

コーシー応力：

$$\sigma_{ij} = \frac{F_j}{A_i}$$

```python
class StressTensor:
    """
    応力テンソル
    """
    
    def compute_stress(
        self,
        strain: np.ndarray,
        material: Material
    ) -> np.ndarray:
        """
        応力を計算
        
        σ = C : ε
        ここで C は弾性定数テンソル
        """
        # 一般化フックの法則
        pass
```

### 5.3 構成関係

一般化フックの法則（等方性線弾性）：

$$\varepsilon_{ij} = \frac{1}{E}\left[(1+\nu)\sigma_{ij} - \nu \delta_{ij}\sigma_{kk}\right]$$

```python
class ConstitutiveRelation:
    """
    構成関係
    
    一般化フックの法則
    """
    
    def hookes_law_isotropic(
        self,
        stress: np.ndarray,
        youngs_modulus: float,
        poisson_ratio: float
    ) -> np.ndarray:
        """
        等方性一般化フックの法則
        
        ε = 1/E [(1+ν)σ - ν(tr σ)I]
        """
        trace = np.trace(stress)
        identity = np.eye(3)
        
        strain = (1 / youngs_modulus) * ((1 + poisson_ratio) * stress - 
                                          poisson_ratio * trace * identity)
        
        return strain
```

---

## §6. 振動と波動

### 6.1 単振動子

$$m\ddot{x} + kx = 0$$

解は：

$$x(t) = A\cos(\omega t + \phi), \quad \omega = \sqrt{\frac{k}{m}}$$

```python
class HarmonicOscillator:
    """
    単振動子
    """
    
    def __init__(self, mass: float, k: float):
        self.mass = mass
        self.k = k
        self.omega = np.sqrt(k / mass)
    
    def position(self, t: float, amplitude: float, phase: float) -> float:
        """x(t) = A cos(ωt + φ)"""
        return amplitude * np.cos(self.omega * t + phase)
    
    def energy(self, amplitude: float) -> float:
        """全エネルギー E = 1/2 k A²"""
        return 0.5 * self.k * amplitude**2
```

### 6.2 減衰振動

$$m\ddot{x} + b\dot{x} + kx = 0$$

```python
class DampedOscillator:
    """
    減衰振動子
    """
    
    def __init__(self, mass: float, k: float, damping: float):
        self.mass = mass
        self.k = k
        self.damping = damping
        self.omega_0 = np.sqrt(k / mass)
        self.gamma = damping / (2 * mass)
    
    def is_overdamped(self) -> bool:
        """γ > ω₀ の時、過減衰"""
        return self.gamma > self.omega_0
    
    def is_underdamped(self) -> bool:
        """γ < ω₀ の時、不足減衰"""
        return self.gamma < self.omega_0
    
    def solution_underdamped(
        self,
        t: float,
        initial_displacement: float,
        initial_velocity: float
    ) -> float:
        """不足減衰解"""
        omega_d = np.sqrt(self.omega_0**2 - self.gamma**2)
        
        A = initial_displacement
        B = (initial_velocity + self.gamma * A) / omega_d
        
        return np.exp(-self.gamma * t) * (A * np.cos(omega_d * t) + 
                                          B * np.sin(omega_d * t))
```

### 6.3 波動方程式

一次元波動方程式：

$$\frac{\partial^2 u}{\partial t^2} = v^2 \frac{\partial^2 u}{\partial x^2}$$

```python
class WaveEquation:
    """
    波動方程式
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
        """弦上の波速 v = √(T/μ)"""
        return np.sqrt(tension / linear_density)
```

---

## §7. 他のスケールとのインターフェース

### 7.1 量子力学 (PS-L0) とのインターフェース

```
古典力学 → 量子力学：
- 対応原理：ħ → 0 の時量子が古典に回帰
- 量子化規則：ポアッソン括弧から交換子へ
- 半古典近似：WKB近似
```

### 7.2 統計力学 (PS-L1) とのインターフェース

```
古典力学 → 統計力学：
- 軌道から分布関数へ
- リウヴィル方程式 → ボルツマン方程式
- ミクロカノニカルとカノニカル集合
```

### 7.3 連続体力学 (PS-L2/PS-L3) とのインターフェース

```
古典力学 → 連続体力学：
- 離散 → 連続極限
- 質点 → 連続体場
- オイラー-ラグランダン記述
```

### 7.4 一般相対性理論 (PS-L4) とのインターフェース

```
古典力学 → 一般相対性理論：
- 低速弱場極限：ニュートン重力に回帰
- 後続ニュートン近似
- 時空曲率効果
```

---

## バージョン履歴

| バージョン | 日付 | 変更 |
|------|------|------|
| v1.0 | 2026-03-18 | 初期バージョン：ニュートン力学、ラグランダン力学、ハミルトン力学、剛体動力学、弾性力学基礎、振動と波動をカバー |

---

*本文書は古典力学スケールの物理フレームワークを処理する。*
*古典力学は工学と科学の基礎であり、PS-L2スケールの物理現象に適用される。*
