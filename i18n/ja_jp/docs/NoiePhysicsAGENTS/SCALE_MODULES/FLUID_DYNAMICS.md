# FLUID_DYNAMICS.md

## 流体力学 (PS-L2, PS-L3)

**スケール：** 10⁻³ ~ 10⁷ m  
**バージョン：** v1.0  
**状態：** 検証済み

---

## 概要

本文書は**流体力学**スケールの物理フレームワークを処理する。NoiePhysicsAGENTS.md §1の物理スケール権限レベル定義に従い、流体力学は液体と気体の運動法則を処理する。

---

## 重要安全・真理プロトコル

> **CRITICAL SAFETY & TRUTH PROTOCOL:**
> 1. AXIOMS.mdのPT-AX11（最小作用量）、PT-AX14（運動量保存）を遵守
> 2. 流体力学は十分に検証された工学的科学である
> 3. 層流と乱流の区別に注意する
> 4. 監査：すべての異常をPHYSICS_AUDIT_TRAILに記録する

---

## 1. 流体の運動学

### 1.1 ラグランジュとオイラーの記述

**オイラー記述**（多用）：

$$\mathbf{v} = \mathbf{v}(\mathbf{x}, t)$$

**物質微分：**

$$\frac{D}{Dt} = \frac{\partial}{\partial t} + \mathbf{v} \cdot \nabla$$

```python
class FluidKinematics:
    """
    流体の運動学
    """
    
    def material_derivative(
        self,
        field: ScalarField,
        velocity: VectorField
    ) -> ScalarField:
        """物質微分の計算"""
        return field.time_derivative() + velocity.dot(field.gradient())
```

---

## 2. Navier-Stokes方程式

### 2.1 完全な方程式系

**連続の式：**

$$\frac{\partial \rho}{\partial t} + \nabla \cdot (\rho \mathbf{v}) = 0$$

**運動量方程式：**

$$\rho \frac{D\mathbf{v}}{Dt} = -\nabla p + \mu \nabla^2 \mathbf{v} + \rho \mathbf{f}$$

**エネルギー方程式：**

$$\rho \frac{D e}{Dt} = -p \nabla \cdot \mathbf{v} + k \nabla^2 T + \Phi$$

```python
class NavierStokesSolver:
    """
    Navier-Stokes方程式ソルバー
    """
    
    def compute_convection(
        self,
        velocity: VectorField,
        field: ScalarField
    ) -> VectorField:
        """対流項の計算 (v·∇)f"""
        return velocity * field.gradient()
    
    def compute_diffusion(
        self,
        field: ScalarField,
        viscosity: float
    ) -> VectorField:
        """拡散項の計算 ν∇²f"""
        return viscosity * field.laplacian()
```

---

## 3. レイノルズ数と流れの状態

### 3.1 レイノルズ数

$$Re = \frac{\rho VL}{\mu} = \frac{VL}{\nu}$$

| Re範囲 | 流れの状態 | 特徴 |
|--------|-----------|------|
| < 2300 | 層流 | 平滑で予測可能 |
| 2300-4000 | 遷移 | 不安定 |
| > 4000 | 乱流 | カオス的、統計的記述 |

### 3.2 レイノルズ応力

乱流では、レイノルズ応力をモデル化する必要がある：

$$-\overline{\rho u_i' u_j'} = \mu_t \left( \frac{\partial \bar{u}_i}{\partial x_j} + \frac{\partial \bar{u}_j}{\partial x_i} \right) - \frac{2}{3} \bar{\rho} k \delta_{ij}$$

---

## 4. 境界層理論

### 4.1 境界層の概念

固体壁の近くでは粘性効果が顕著である：

$$\delta \sim \frac{L}{\sqrt{Re}}$$

```python
class BoundaryLayer:
    """
    境界層
    """
    
    def compute_boundary_layer_thickness(
        self,
        reynolds_number: float,
        length_scale: float
    ) -> float:
        """境界層厚さの計算"""
        return length_scale / np.sqrt(reynolds_number)
```

---

## 5. 数値計算法

### 5.1 離散化手法

| 手法 | 利点 | 欠点 |
|------|------|------|
| 有限差分法 | シンプル | 幾何学的柔軟性が低い |
| 有限体積法 | 保存性 | 精度が限定的 |
| 有限要素法 | 幾何学的柔軟性 | 計算量大 |
| スペクトル法 | 高精度 | 周期境界 |

### 5.2 求解戦略

```python
class FluidSolver:
    """
    流体ソルバー
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
        非圧縮性流体の求解
        
        方法：SIMPLE, PISO, 射影法
        """
        pass
```

---

## 6. 多相流

### 6.1 相境界面

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
        """表面張力の計算"""
        # F = σ ∫ κ n dA
        pass
```

---

## 7. 他のスケールとのインターフェース

### 7.1 連続体力学とのインターフェース

```
連続体力学 → 流体力学：
- 応力テンソルの簡略化
- ニュートン流体の構成方程式
```

### 7.2 プラズマ物理学とのインターフェース

```
流体力学 → MHD：
- 導電性流体の特殊な場合
```

---

*本文書は流体力学スケールの物理フレームワークを処理する。*
*流体力学は航空、船舶、能源などの工学の基盤である。*
