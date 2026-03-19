# FLUID_DYNAMICS.md

## 流體動力學 (PS-L2, PS-L3)

**尺度：** 10⁻³ ~ 10⁷ m  
**版本：** v1.0  
**狀態：** 驗證性

---

## 概述

本文檔處理**流體動力學**尺度的物理框架。根據 NoiePhysicsAGENTS.md §1 的物理尺度權限層級定義，流體動力學處理液體和氣體的運動規律。

---

## 關鍵安全與真理協議

> **CRITICAL SAFETY & TRUTH PROTOCOL:**
> 1. 遵守 AXIOMS.md 的 PT-AX11 (最小作用量)、PT-AX14 (動量守恆)
> 2. 流體動力學是經過充分驗證的工程科學
> 3. 注意層流與亂流的區分
> 4. 審計：將所有異常記錄至 PHYSICS_AUDIT_TRAIL

---

## 1. 流體運動學

### 1.1 拉格朗日與歐拉描述

**歐拉描述**（常用）：

$$\mathbf{v} = \mathbf{v}(\mathbf{x}, t)$$

**物質導數**：

$$\frac{D}{Dt} = \frac{\partial}{\partial t} + \mathbf{v} \cdot \nabla$$

```python
class FluidKinematics:
    """
    流體運動學
    """
    
    def material_derivative(
        self,
        field: ScalarField,
        velocity: VectorField
    ) -> ScalarField:
        """計算物質導數"""
        return field.time_derivative() + velocity.dot(field.gradient())
```

---

## 2. Navier-Stokes 方程

### 2.1 完整方程組

**連續方程**：

$$\frac{\partial \rho}{\partial t} + \nabla \cdot (\rho \mathbf{v}) = 0$$

**動量方程**：

$$\rho \frac{D\mathbf{v}}{Dt} = -\nabla p + \mu \nabla^2 \mathbf{v} + \rho \mathbf{f}$$

**能量方程**：

$$\rho \frac{D e}{Dt} = -p \nabla \cdot \mathbf{v} + k \nabla^2 T + \Phi$$

```python
class NavierStokesSolver:
    """
    Navier-Stokes 方程求解器
    """
    
    def compute_convection(
        self,
        velocity: VectorField,
        field: ScalarField
    ) -> VectorField:
        """計算對流項 (v·∇)f"""
        return velocity * field.gradient()
    
    def compute_diffusion(
        self,
        field: ScalarField,
        viscosity: float
    ) -> VectorField:
        """計算擴散項 ν∇²f"""
        return viscosity * field.laplacian()
```

---

## 3. 雷諾數與流態

### 3.1 雷諾數

$$Re = \frac{\rho VL}{\mu} = \frac{VL}{\nu}$$

| Re 範圍 | 流態 | 特徵 |
|---------|------|------|
| < 2300 | 層流 | 平滑、可預測 |
| 2300-4000 | 過渡 | 不穩定 |
| > 4000 | 亂流 | 混沌、統計描述 |

### 3.2 雷諾應力

在亂流中，需要對雷諾應力進行模型化：

$$-\overline{\rho u_i' u_j'} = \mu_t \left( \frac{\partial \bar{u}_i}{\partial x_j} + \frac{\partial \bar{u}_j}{\partial x_i} \right) - \frac{2}{3} \bar{\rho} k \delta_{ij}$$

---

## 4. 邊界層理論

### 4.1 邊界層概念

在固體邊界附近，粘性效應顯著：

$$\delta \sim \frac{L}{\sqrt{Re}}$$

```python
class BoundaryLayer:
    """
    邊界層
    """
    
    def compute_boundary_layer_thickness(
        self,
        reynolds_number: float,
        length_scale: float
    ) -> float:
        """計算邊界層厚度"""
        return length_scale / np.sqrt(reynolds_number)
```

---

## 5. 數值方法

### 5.1 離散化方法

| 方法 | 優點 | 缺點 |
|------|------|------|
| 有限差分 | 簡單 | 幾何靈活性差 |
| 有限體積 | 守恆 | 精度有限 |
| 有限元 | 幾何靈活 | 計算量大 |
| 譜方法 | 高精度 | 週期邊界 |

### 5.2 求解策略

```python
class FluidSolver:
    """
    流體求解器
    """
    
    def solve_incompressible(
        self,
        domain: Mesh,
        initial_condition: Field,
        boundary_conditions: BC,
        time_step: float,
        num_steps: int
    ) -> Solution:
        """
        求解不可壓縮流
        
        方法：SIMPLE, PISO, 投影法
        """
        pass
```

---

## 6. 多相流

### 6.1 相界面

```python
class MultiPhaseFlow:
    """
    多相流
    """
    
    def compute_surface_tension_force(
        self,
        interface: Interface,
        surface_tension: float
    ) -> VectorField:
        """計算表面張力"""
        # F = σ ∫ κ n dA
        pass
```

---

## 7. 與其他尺度的接口

### 7.1 與連續介質力學的接口

```
連續介質力學 → 流體動力學：
- 應力張量簡化
- 牛頓流體本構方程
```

### 7.2 與電漿物理的接口

```
流體動力學 → MHD：
- 導電流體的特殊情況
```

---

*本文檔處理流體動力學尺度的物理框架。*
*流體動力學是航空、船舶、能源等工程的基礎。*
