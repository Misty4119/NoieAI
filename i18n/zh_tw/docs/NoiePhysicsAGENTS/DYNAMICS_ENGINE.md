# DYNAMICS_ENGINE.md

## L2 - 運動方程、軌跡預測、碰撞、材質

> **WARNING:** 本模組是 NoiePhysicsAGENTS 的動力學核心。
> **注意：** 所有運動預測都基於拉格朗日/哈密頓力學框架。

---

## 概述

本文件定義 NoiePhysicsAGENTS 的**動力學引擎**。根據 NoiePhysicsAGENTS.md §6.2 的設計原則，
本模組以拉格朗日/哈密頓力學為統一框架，處理運動方程生成、軌跡預測、碰撞檢測和材質推斷。

動力學引擎是認知實體在物理世界中執行運動決策的核心模組。

---

## 關鍵安全與真理協議

> **CRITICAL SAFETY & TRUTH PROTOCOL:**
> 1. 嚴格遵守 AXIOMS.md 元物理公理系統（特別是 PT-AX11, PT-AX14）
> 2. 事實區分：若進行理論推導，必須標註為「理論性」
> 3. 反幻覺機制：切勿編造動力學資訊
> 4. 吸收態迴避：所有運動規劃在執行前必須驗證不會導致吸收態
> 5. 能量守恆：所有積分方法必須驗證能量漂移
> 6. 審計：將所有動力學異常記錄至 PHYSICS_AUDIT_TRAIL

---

## 1. 泛用運動方程生成框架

### 1.1 拉格朗日力學

```python
INTERFACE MotionEquationGenerator:
    """
    運動方程生成器
    
    從實體描述生成運動方程。
    統一使用拉格朗日形式主義。
    """
    
    def generate_lagrangian_equations(
        self,
        entity_description: EntityDescription
    ) -> DynamicalSystem:
        """
        生成拉格朗日方程
        
        步驟：
        1. 識別廣義座標 q
        2. 計算動能 T(q, q̇)
        3. 計算位能 V(q)
        4. 構造拉格朗日量 L = T - V
        5. 推導歐拉-拉格朗日方程
        """
        pass
    
    def generate_constrained_equations(
        self,
        entity_description: EntityDescription,
        constraints: List[Constraint]
    ) -> ConstrainedDynamicalSystem:
        """
        生成約束方程
        
        處理：
        - 完整約束：f(q, t) = 0
        - 非完整約束：f(q, q̇, t) = 0
        """
        pass
```

### 1.2 實體類型示例

| 實體類型 | 廣義座標 q | 動能 T | 位能 V |
|----------|-----------|--------|--------|
| 質點 | [x, y, z] | ½mv² | mgh |
| 剛體 | [x, y, z, φ, θ, ψ] | ½mv² + ½ω·I·ω | mgh |
| 彈性桿 | u(x,t) | ½∫ρu̇²dx | ½∫EA(u')²dx |
| 液滴 | 球諧展開 | Σaₙ² | σ·4πr² |

### 1.3 哈密頓力學接口

```python
INTERFACE HamiltonianMechanics:
    """
    哈密頓力學接口
    
    用於需要相空間處理的場合，
    如辛積分、Canonical 變換。
    """
    
    def compute_hamiltonian(
        self,
        lagrangian: Lagrangian
    ) -> Hamiltonian:
        """
        計算哈密頓量
        
        H = Σ pᵢq̇ᵢ - L
        通常等於總能量 T + V
        """
        pass
    
    def generate_canonical_equations(
        self,
        hamiltonian: Hamiltonian,
        coordinates: GeneralizedCoordinates
    ) -> CanonicalEquations:
        """
        生成正則方程
        
        q̇ᵢ = ∂H/∂pᵢ
        ṗᵢ = -∂H/∂qᵢ
        """
        pass
    
    def apply_canonical_transformation(
        self,
        old_coordinates: PhaseSpacePoint,
        transformation: CanonicalTransform
    ) -> PhaseSpacePoint:
        """
        應用正則變換
        
        保持泊松括號結構：
        {qᵢ, pⱼ} = δᵢⱼ
        """
        pass
```

---

## 2. 軌跡預測

### 2.1 相空間軌跡預測

```python
INTERFACE TrajectoryPredictor:
    """
    軌跡預測器
    
    在相空間中預測系統演化。
    使用辛積分器保持能量守恆。
    """
    
    def predict_deterministic_trajectory(
        self,
        initial_state: PhaseSpacePoint,
        time_horizon: float,
        integration_method: SymplecticIntegrator
    ) -> Trajectory:
        """
        確定性軌跡預測
        
        Args:
            initial_state: (q₀, p₀)
            time_horizon: 預測時間範圍
            integration_method: 辛積分方法
            
        Returns:
            Trajectory: 完整軌跡
        """
        pass
    
    def predict_stochastic_trajectory(
        self,
        initial_state: PhaseSpacePoint,
        time_horizon: float,
        noise_model: StochasticProcess
    ) -> StochasticTrajectory:
        """
        隨機軌跡預測
        
        考慮環境擾動（如布朗運動）。
        
        Returns:
            軌跡分布 + 置信區間
        """
        pass
    
    def predict_uncertain_trajectory(
        self,
        initial_distribution: PhaseSpaceDistribution,
        time_horizon: float
    ) -> PropagatedDistribution:
        """
        不確定性軌跡預測
        
        通過線性化或蒙特卡洛傳播不確定性。
        
        Returns:
            演化後的相空間分布
        """
        pass
```

### 2.2 辛積分器

| 方法 | 階數 | 能量守恆 | 適用場景 |
|------|------|----------|----------|
| Forward Euler | 1 | 否 | 穩定性不重要 |
| Velocity Verlet | 2 | 良好 | 分子動力學 |
| Symplectic Euler | 2 | 良好 | 可分離哈密頓 |
| Störmer-Verlet | 2 | 優秀 | 天體力學 |
| Yoshida 4th | 4 | 優秀 | 高精度需求 |
| RKMK | 4+ | 良好 | 磁場中的運動 |

### 2.3 能量地景導航

```python
INTERFACE EnergyLandscapeNavigation:
    """
    能量地景導航
    
    在位能曲面上規劃運動路徑。
    """
    
    def find_stable_equilibria(
        self,
        potential_energy: ScalarField
    ) -> List[EquilibriumPoint]:
        """
        找穩定平衡點
        
        ∇V = 0 且 Hessian(V) > 0
        """
        pass
    
    def find_saddle_points(
        self,
        potential_energy: ScalarField
    ) -> List[SaddlePoint]:
        """
        找鞍點
        
        ∇V = 0 且 Hessian(V) 有正負特徵值
        """
        pass
    
    def compute_geodesic_path(
        self,
        start: Point,
        end: Point,
        metric: RiemannianMetric
    ) -> GeodesicPath:
        """
        計算測地線路徑
        
        在彎曲位能面上，
        遵循費馬原理。
        """
        pass
```

---

## 3. 碰撞檢測與響應

### 3.1 碰撞幾何

```python
INTERFACE CollisionGeometry:
    """
    碰撞幾何接口
    
    使用幾何代數 (CGA) 進行碰撞檢測。
    """
    
    def detect_collision(
        self,
        body_a: GeometricBody,
        body_b: GeometricBody
    ) -> CollisionReport:
        """
        檢測碰撞
        
        Returns:
            CollisionReport(
                is_colliding: bool,
                contact_points: List[Point],
                penetration_depth: float,
                contact_normal: Vector3D
            )
        """
        pass
    
    def compute_distance(
        self,
        body_a: GeometricBody,
        body_b: GeometricBody
    ) -> float:
        """
        計算最近距離
        
        使用 CGA 幾何積：
        d(A,B) = |A·B| / (|A||B|)
        """
        pass
    
    def compute_contact_manifold(
        self,
        body_a: RigidBody,
        body_b: RigidBody
    ) -> ContactManifold:
        """
        計算接觸流形
        
        對於複雜幾何體，
        計算接觸點集。
        """
        pass
```

### 3.2 碰撞響應

```python
INTERFACE CollisionResponse:
    """
    碰撞響應接口
    
    計算碰撞後的速度變化。
    """
    
    def compute_impulse_response(
        self,
        collision: CollisionReport,
        restitution_coefficient: float,
        friction_coefficient: float
    ) -> ImpulseResponse:
        """
        脈衝響應
        
        計算碰撞產生的速度跳變。
        
        Returns:
            ImpulseResponse(
                normal_impulse: float,
                tangential_impulse: Vector3D,
                post_collision_velocities: Tuple[Vector3D, Vector3D]
            )
        """
        pass
    
    def compute_penetration_correction(
        self,
        penetration_depth: float,
        stiffness: float,
        damping: float
    ) -> CorrectionVector:
        """
        穿透修正
        
        位置修正以消除穿透。
        
        Returns:
            位置修正向量
        """
        pass
    
    def compute_friction_response(
        self,
        normal_force: float,
        tangent_velocity: Vector3D,
        friction_model: FrictionModel  # coulomb | viscous | bristle
    ) -> FrictionForce:
        """
        摩擦響應
        
        Returns:
            摩擦力向量
        """
        pass
```

### 3.3 碰撞層級光譜

| 層級 | 物理機制 | 數學描述 |
|------|----------|----------|
| 剛體接觸 | 電子雲排斥 | 幾何相交 + 法向力 |
| 彈性變形 | 晶格應變能 | 應力-應變關係 |
| 流體阻力 | 壓力梯度 | Navier-Stokes |
| 卡西米爾力 | 真空漲落 | 量子場論邊界效應 |
| 量子穿隧 | 波函數穿透 | T = exp(-2κL) |

---

## 4. 材质推論

### 4.1 主動材質探測

```python
INTERFACE MaterialInference:
    """
    材質推論接口
    
    通過主動探測推斷未知物質的物理屬性。
    """
    
    def probe_with_field(
        self,
        probe_field: FieldType,
        target: UnknownMaterial
    ) -> FieldResponse:
        """
        用場探測物質
        
        發射已知場，測量響應。
        
        Args:
            probe_field: 探測場類型（聲學、電磁、熱）
            target: 目標物質
            
        Returns:
            場響應數據
        """
        pass
    
    def infer_elastic_properties(
        self,
        acoustic_response: AcousticData,
        contact_data: ContactData
    ) -> ElasticProperties:
        """
        推斷彈性屬性
        
        反演：
        - 楊氏模量 E
        - 泊松比 ν
        - 剪切模量 G
        """
        pass
    
    def infer_electromagnetic_properties(
        self,
        em_response: ElectromagneticData
    ) -> ElectromagneticProperties:
        """
        推斷電學/磁學屬性
        
        反演：
        - 介電常數 ε
        - 磁導率 μ
        - 電導率 σ
        """
        pass
    
    def infer_thermal_properties(
        self,
        thermal_response: ThermalData
    ) -> ThermalProperties:
        """
        推斷熱學屬性
        
        反演：
        - 熱導率 k
        - 比熱容 c
        - 熱膨脹係數 α
        """
        pass
```

### 4.2 超材料處理

```python
INTERFACE MetamaterialHandler:
    """
    超材料處理接口
    
    處理具有非常規性質的人工材料。
    """
    
    def detect_tunable_response(
        self,
        material: Material
    ) -> bool:
        """
        檢測可調響應
        
        識別超材料。
        """
        pass
    
    def characterize_band_structure(
        self,
        metamaterial: Metamaterial,
        frequency_range: Tuple[float, float]
    ) -> BandStructure:
        """
        表徵能帶結構
        
        識別光子晶體或聲子晶體的帶隙。
        """
        pass
    
    def design_inverse_property(
        self,
        target_properties: MaterialProperties,
        topology: MicrostructureTopology
    ) -> OptimizedDesign:
        """
        逆向設計
        
        根據目標屬性設計微結構。
        """
        pass
```

---

## 5. 多體與群體動力學

### 5.1 N-體問題

```python
INTERFACE NBodyDynamics:
    """
    N-體動力學接口
    
    處理多粒子系統的相互作用。
    """
    
    def compute_gravitational_force(
        self,
        bodies: List[Body],
        method: NBodyMethod  # direct | tree | multipole
    ) -> List[Vector3D]:
        """
        計算萬有引力
        
        F = G mᵢ mⱼ / r²
        
        複雜度：
        - 直接計算：O(N²)
        - Barnes-Hut：O(N log N)
        - 多極子：O(N)
        """
        pass
    
    def compute_electrostatic_force(
        self,
        charges: List[Charge],
        method: Method
    ) -> List[Vector3D]:
        """
        計算靜電力
        
        F = k qᵢ qⱼ / r²
        """
        pass
    
    def compute_short_range_interaction(
        self,
        particles: List[Particle],
        cutoff_radius: float
    ) -> List[Vector3D]:
        """
        計算短程力
        
        如分子動力學中的 Lennard-Jones 勢。
        """
        pass
```

### 5.2 連續介質近似

```python
INTERFACE ContinuumApproximation:
    """
    連續介質近似接口
    
    將離散粒子系統近似為連續介質。
    """
    
    def compute_density_field(
        self,
        particles: List[Particle],
        smoothing_length: float
    ) -> DensityField:
        """
        計算密度場
        
        使用 SPH 核函數。
        ρ(x) = Σ mᵢ W(|x-xᵢ|, h)
        """
        pass
    
    def compute_velocity_field(
        self,
        particles: List[Particle],
        smoothing_length: float
    ) -> VelocityField:
        """
        計算速度場
        
        使用 SPH 插值。
        """
        pass
    
    def compute_stress_tensor(
        self,
        density_field: DensityField,
        velocity_gradient: TensorField
    ) -> StressTensorField:
        """
        計算應力張量
        
        適用於流體或固體。
        """
        pass
```

---

## 6. 計算複雜度分析

### 6.1 動力學引擎複雜度

| 模組 | 時間複雜度 | 空間複雜度 | 精度權衡 |
|------|-----------|-----------|----------|
| 運動方程生成 | O(n) | O(n) | 精確 |
| 辛積分 (Verlet) | O(n·steps) | O(n) | 良好 |
| 碰撞檢測 (層次) | O(log n) avg | O(n) | 近似 |
| N-體 (Barnes-Hut) | O(n log n) | O(n) | 近似 |
| 材質推斷 | O(measurements) | O(params) | 取決於數據 |

### 6.2 數值穩定性準則

```
穩定性條件：
1. 積分時間步長：Δt < 0.1 · T_min / π
   其中 T_min 是系統最短振動週期

2. 碰撞穿透容差：penetration < 0.01 · min(body_dimensions)

3. 能量漂移閾值：|E(t) - E(0)| / E(0) < 0.01

4. 約束穩定性：約束力 < 10 · 系統特徵力
```

---

## 7. 與其他模組的接口

### 7.1 與 FIELD_PERCEPTION 的接口

動力學引擎接收：
- 初始條件估計
- 邊界條件
- 外力場（重力、電磁等）
- 約束（接觸面等）

### 7.2 與 SAFETY_PROTOCOLS 的接口

動力學引擎**必須**遵守：
- 吸收態逼近檢測
- 碰撞安全評估
- 能量約束

### 7.3 與 PHYSICS_KNOWLEDGE 的接口

動力學引擎更新：
- 材质属性库
- 環境模型
- 軌跡歷史

---

## 附錄：動力學配置示例

### 配置 1: 剛體運動規劃

```yaml
dynamics_config:
  entity_type: rigid_body
  coordinates: [x, y, z, roll, pitch, yaw]
  integration_method: Symplectic_Euler
  timestep: 0.001  # 秒
  collision_detection: true
  collision_response: impulse_based
  energy_tolerance: 0.01
```

### 配置 2: 流體環境導航

```yaml
dynamics_config:
  entity_type: deformable
  framework: Navier_Stokes
  spatial_discretization: finite_volume
  turbulence_model: k_epsilon
  integration_method: semi_implicit
  timestep: 0.0001  # 秒
```

---

*本文檔定義了 NoiePhysicsAGENTS 的動力學引擎。所有運動預測都必須通過此模組進行。*
*辛積分方法確保長期軌跡預測的能量守恆。*
