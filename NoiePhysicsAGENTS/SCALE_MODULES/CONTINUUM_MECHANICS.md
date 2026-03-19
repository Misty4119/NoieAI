# CONTINUUM_MECHANICS.md

## 連續介質力學 (PS-L2, PS-L3)

**尺度：** 10⁻³ ~ 10⁷ m  
**版本：** v1.0  
**狀態：** 驗證性

---

## 概述

本文檔處理**連續介質力學**尺度的物理框架。根據 NoiePhysicsAGENTS.md §1 的物理尺度權限層級定義，PS-L2 和 PS-L3 分別代表人類/古典尺度和地球/地質尺度，涵蓋從毫米到數千公里的物理現象。

---

## 關鍵安全與真理協議

> **CRITICAL SAFETY & TRUTH PROTOCOL:**
> 1. 遵守 AXIOMS.md 的 PT-AX11 (最小作用量)、PT-AX14 (動量守恆)
> 2. 連續介質力學是經過充分驗證的工程科學
> 3. 注意連續介質假設的有效範圍
> 4. 審計：將所有異常記錄至 PHYSICS_AUDIT_TRAIL

---

## 1. 連續介質假設

### 1.1 基本假設

連續介質力學假設物質可以視為連續分佈的質點：

```
┌─────────────────────────────────────────────────────────┐
│              連續介質 vs 離散粒子                          │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  離散視圖：                                              │
│  ○ ○ ○ ○ ○ ○ ○ ○ ○                                    │
│  ○ ○ ○ ○ ○ ○ ○ ○ ○  ← 原子/分子                      │
│  ○ ○ ○ ○ ○ ○ ○ ○ ○                                    │
│                                                         │
│  連續視圖：                                              │
│  ═══════════════════════════════                        │
│  ρ(x,y,z) ← 密度場                                      │
│  v(x,y,z) ← 速度場                                      │
│                                                         │
│  有效性：特徵尺度 >> 分子間距                             │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 1.2 運動學描述

**拉格朗日描述**（物質坐標）：

$$\mathbf{x} = \mathbf{x}(\mathbf{X}, t)$$

**歐拉描述**（空間坐標）：

$$\mathbf{v} = \mathbf{v}(\mathbf{x}, t)$$

```python
class KinematicsDescription:
    """
    運動學描述
    """
    
    def lagrangian_to_eulerian(
        self,
        field_lagrangian: Field,
        deformation_gradient: Tensor
    ) -> Field:
        """拉格朗日 → 歐拉"""
        pass
    
    def compute_deformation_gradient(
        self,
        initial_position: Vector3D,
        current_position: Vector3D
    ) -> Tensor:
        """計算變形梯度"""
        # F = ∂x/∂X
        pass
```

---

## 2. 應力與應變

### 2.1 應變張量

**格林-拉格朗日應變**：

$$E_{ij} = \frac{1}{2}\left(\frac{\partial u_i}{\partial X_j} + \frac{\partial u_j}{\partial X_i} + \frac{\partial u_k}{\partial X_i}\frac{\partial u_k}{\partial X_j}\right)$$

**阿爾曼西應變**（線性）：

$$\varepsilon_{ij} = \frac{1}{2}\left(\frac{\partial u_i}{\partial x_j} + \frac{\partial u_j}{\partial x_i}\right)$$

```python
class StrainTensor:
    """
    應變張量
    """
    
    def compute_linear_strain(
        self,
        displacement_gradient: Tensor
    ) -> Tensor:
        """計算線性應變"""
        return 0.5 * (displacement_gradient + displacement_gradient.T)
    
    def compute_green_lagrange(
        self,
        displacement_gradient: Tensor
    ) -> Tensor:
        """計算格林-拉格朗日應變"""
        return 0.5 * (displacement_gradient + displacement_gradient.T + 
                      displacement_gradient.T @ displacement_gradient)
```

### 2.2 應力張量

**柯西應力**（真實應力）：

$$\sigma_{ij} = \lim_{\Delta A_j \to 0} \frac{\Delta F_i}{\Delta A_j}$$

```python
class StressTensor:
    """
    應力張量
    """
    
    def compute_cauchy_stress(
        self,
        force: Vector3D,
        area: Vector3D
    ) -> Tensor:
        """計算柯西應力"""
        return force.outer(area.normalized()) / area.magnitude()
```

### 2.3 應力-應變關係

**虎克定律**（各向同性線彈性）：

$$\sigma_{ij} = \lambda \varepsilon_{kk}\delta_{ij} + 2\mu\varepsilon_{ij}$$

其中 $\lambda$ 和 $\mu$ 為拉梅常數。

```python
class ConstitutiveRelation:
    """
    本構關係
    """
    
    def hooke_isotropic(
        self,
        strain: Tensor,
        youngs_modulus: float,
        poisson_ratio: float
    ) -> Tensor:
        """各向同性虎克定律"""
        lam = youngs_modulus * poisson_ratio / ((1 + poisson_ratio) * (1 - 2 * poisson_ratio))
        mu = youngs_modulus / (2 * (1 + poisson_ratio))
        
        trace = np.trace(strain)
        return lam * trace * np.eye(3) + 2 * mu * strain
```

---

## 3. 運動方程

### 3.1 平衡方程

**動量平衡**：

$$\rho \frac{Dv_i}{Dt} = \frac{\partial \sigma_{ij}}{\partial x_j} + \rho b_i$$

**質量守恆**（連續方程）：

$$\frac{D\rho}{Dt} + \rho \frac{\partial v_i}{\partial x_i} = 0$$

**能量守恆**：

$$\rho \frac{De}{Dt} = \sigma_{ij}\frac{\partial v_i}{\partial x_j} + \frac{\partial q_i}{\partial x_i} + \rho r$$

```python
class ConservationLaws:
    """
    守恆定律
    """
    
    def momentum_equation(
        self,
        density: float,
        velocity: Vector3D,
        stress: Tensor,
        body_force: Vector3D
    ) -> Vector3D:
        """動量方程"""
        divergence_stress = stress.divergence()
        return density * velocity.time_derivative() - divergence_stress - density * body_force
    
    def continuity_equation(
        self,
        density: float,
        velocity: Vector3D
    ) -> float:
        """連續方程"""
        return density.time_derivative() + density * velocity.divergence()
```

---

## 4. 彈性力學

### 4.1 邊界值問題

```python
class ElasticBoundaryValueProblem:
    """
    彈性邊界值問題
    """
    
    def solve_displacement(
        self,
        domain: Domain,
        boundary_conditions: BoundaryConditions,
        material_properties: MaterialProperties
    ) -> DisplacementField:
        """
        求解位移場
        
        方法：有限元、邊界元、解析解
        """
        pass
```

### 4.2 邊界條件

| 類型 | 數學表達 | 物理意義 |
|------|----------|----------|
| 位移邊界 | u = ū | 固定支撐 |
| 力邊界 | σ·n = t̄ | 載荷 |
| 混合邊界 | 組合 | 彈性支撐 |

---

## 5. 塑性力學

### 5.1 屈服準則

**馮·米塞斯屈服準則**：

$$f(\sigma) = \sqrt{\frac{1}{2}(\sigma_1-\sigma_2)^2 + (\sigma_2-\sigma_3)^2 + (\sigma_3-\sigma_1)^2} - \sigma_y = 0$$

**特雷斯科屈服準則**：

$$f(\sigma) = \max(|\sigma_1-\sigma_2|, |\sigma_2-\sigma_3|, |\sigma_3-\sigma_1|) - \sigma_y = 0$$

### 5.2 流動理論

```python
class PlasticFlow:
    """
    塑性流動
    """
    
    def compute_plastic_strain_increment(
        self,
        stress: Tensor,
        yield_function: YieldFunction,
        hardening_law: HardeningLaw
    ) -> Tensor:
        """計算塑性應變增量"""
        pass
```

---

## 6. 有限變形理論

### 6.1 有限變形的挑戰

當變形較大時，需要使用更精確的描述：

- 應變度量選擇
- 客觀率（Jaumann, Green-Naghdi）
- 穩定性

### 6.2 本構方程

```python
class FiniteDeformation:
    """
    有限變形理論
    """
    
    def compute_second_piola_kirchhoff(
        self,
        deformation_gradient: Tensor,
        strain_energy: Callable
    ) -> Tensor:
        """計算第二 P-K 應力"""
        # S = ∂W/∂E
        pass
```

---

## 7. 與其他尺度的接口

### 7.1 與統計力學 (PS-L1) 的接口

```
統計力學 → 連續介質力學：
- 從分子運動論湧現
- 巨觀參數的微觀基礎
```

### 7.2 與流體動力學的接口

```
連續介質力學 → 流體動力學：
- 流動是特殊的連續介質
- 應力張量簡化為壓力
```

---

*本文檔處理連續介質力學尺度的物理框架。*
*連續介質力學是工程科學的基礎，應用廣泛。*
