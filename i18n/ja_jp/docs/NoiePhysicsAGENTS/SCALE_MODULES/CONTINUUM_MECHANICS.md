# CONTINUUM_MECHANICS.md

## 連続体力学 (PS-L2, PS-L3)

**スケール：** 10⁻³ ~ 10⁷ m  
**バージョン：** v1.0  
**状態：** 検証済み

---

## 概要

本文書は**連続体力学**スケールの物理フレームワークを処理する。NoiePhysicsAGENTS.md §1の物理スケール権限レベル定義に従い、PS-L2とPS-L3はそれぞれ人間/古典スケールと地球/地質スケールを表し、ミリメートルから数千キロメートルまでの物理現象をカバーする。

---

## 重要安全・真理プロトコル

> **CRITICAL SAFETY & TRUTH PROTOCOL:**
> 1. AXIOMS.mdのPT-AX11（最小作用量）、PT-AX14（運動量保存）を遵守
> 2. 連続体力学は十分に検証された工学的科学である
> 3. 連続体仮定の有効範囲に注意する
> 4. 監査：すべての異常をPHYSICS_AUDIT_TRAILに記録する

---

## 1. 連続体仮定

### 1.1 基本仮定

連続体力学では、物質は連続的に分布する質点として扱う：

```
┌─────────────────────────────────────────────────────────┐
│              連続体 vs 離散粒子                          │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  離散的ビュー：                                         │
│  ○ ○ ○ ○ ○ ○ ○ ○ ○                                    │
│  ○ ○ ○ ○ ○ ○ ○ ○ ○  ← 原子/分子                     │
│  ○ ○ ○ ○ ○ ○ ○ ○ ○                                    │
│                                                         │
│  連続的ビュー：                                         │
│  ═══════════════════════════════                        │
│  ρ(x,y,z) ← 密度場                                     │
│  v(x,y,z) ← 速度場                                     │
│                                                         │
│  有効性：特徴スケール >> 分子間距離                      │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 1.2 運動学的記述

**ラグランジュ記述**（物質座標）：

$$\mathbf{x} = \mathbf{x}(\mathbf{X}, t)$$

**オイラー記述**（空間座標）：

$$\mathbf{v} = \mathbf{v}(\mathbf{x}, t)$$

```python
class KinematicsDescription:
    """
    運動学的記述
    """
    
    def lagrangian_to_eulerian(
        self,
        field_lagrangian: Field,
        deformation_gradient: Tensor
    ) -> Field:
        """ラグランジュ → オイラー"""
        pass
    
    def compute_deformation_gradient(
        self,
        initial_position: Vector3D,
        current_position: Vector3D
    ) -> Tensor:
        """変形勾配の計算"""
        # F = ∂x/∂X
        pass
```

---

## 2. 応力とひずみ

### 2.1 ひずみテンソル

**グリーン・ラグランジュひずみ：**

$$E_{ij} = \frac{1}{2}\left(\frac{\partial u_i}{\partial X_j} + \frac{\partial u_j}{\partial X_i} + \frac{\partial u_k}{\partial X_i}\frac{\partial u_k}{\partial X_j}\right)$$

**アルマンシひずみ**（線形）：

$$\varepsilon_{ij} = \frac{1}{2}\left(\frac{\partial u_i}{\partial x_j} + \frac{\partial u_j}{\partial x_i}\right)$$

```python
class StrainTensor:
    """
    ひずみテンソル
    """
    
    def compute_linear_strain(
        self,
        displacement_gradient: Tensor
    ) -> Tensor:
        """線形ひずみの計算"""
        return 0.5 * (displacement_gradient + displacement_gradient.T)
    
    def compute_green_lagrange(
        self,
        displacement_gradient: Tensor
    ) -> Tensor:
        """グリーン・ラグランジュひずみの計算"""
        return 0.5 * (displacement_gradient + displacement_gradient.T + 
                      displacement_gradient.T @ displacement_gradient)
```

### 2.2 応力テンソル

**コーシー応力**（真応力）：

$$\sigma_{ij} = \lim_{\Delta A_j \to 0} \frac{\Delta F_i}{\Delta A_j}$$

```python
class StressTensor:
    """
    応力テンソル
    """
    
    def compute_cauchy_stress(
        self,
        force: Vector3D,
        area: Vector3D
    ) -> Tensor:
        """コーシー応力の計算"""
        return force.outer(area.normalized()) / area.magnitude()
```

### 2.3 応力-ひずみ関係

**フックの法則**（等方性線形弾性）：

$$\sigma_{ij} = \lambda \varepsilon_{kk}\delta_{ij} + 2\mu\varepsilon_{ij}$$

ただし $\lambda$ と $\mu$ はラメ定数である。

```python
class ConstitutiveRelation:
    """
    構成関係
    """
    
    def hooke_isotropic(
        self,
        strain: Tensor,
        youngs_modulus: float,
        poisson_ratio: float
    ) -> Tensor:
        """等方性フックの法則"""
        lam = youngs_modulus * poisson_ratio / ((1 + poisson_ratio) * (1 - 2 * poisson_ratio))
        mu = youngs_modulus / (2 * (1 + poisson_ratio))
        
        trace = np.trace(strain)
        return lam * trace * np.eye(3) + 2 * mu * strain
```

---

## 3. 運動方程式

### 3.1 平衡方程式

**運動量保存：**

$$\rho \frac{Dv_i}{Dt} = \frac{\partial \sigma_{ij}}{\partial x_j} + \rho b_i$$

**質量保存**（連続の式）：

$$\frac{D\rho}{Dt} + \rho \frac{\partial v_i}{\partial x_i} = 0$$

**エネルギー保存：**

$$\rho \frac{De}{Dt} = \sigma_{ij}\frac{\partial v_i}{\partial x_j} + \frac{\partial q_i}{\partial x_i} + \rho r$$

```python
class ConservationLaws:
    """
    保存法則
    """
    
    def momentum_equation(
        self,
        density: float,
        velocity: Vector3D,
        stress: Tensor,
        body_force: Vector3D
    ) -> Vector3D:
        """運動量方程式"""
        divergence_stress = stress.divergence()
        return density * velocity.time_derivative() - divergence_stress - density * body_force
    
    def continuity_equation(
        self,
        density: float,
        velocity: Vector3D
    ) -> float:
        """連続の式"""
        return density.time_derivative() + density * velocity.divergence()
```

---

## 4. 弾性力学

### 4.1 境界値問題

```python
class ElasticBoundaryValueProblem:
    """
    弾性境界値問題
    """
    
    def solve_displacement(
        self,
        domain: Domain,
        boundary_conditions: BoundaryConditions,
        material_properties: MaterialProperties
    ) -> DisplacementField:
        """
        変位場の求解
        
        方法：有限要素法、境界要素法、解析解
        """
        pass
```

### 4.2 境界条件

| 種類 | 数学的表現 | 物理的意味 |
|------|-----------|-----------|
| 変位境界 | u = ū | 固定支持 |
| 力境界 | σ·n = t̄ | 荷重 |
| 混合境界 | 組み合わせ | 弾性支持 |

---

## 5. 塑性力学

### 5.1 降伏条件

**フォン・ミーゼス降伏条件：**

$$f(\sigma) = \sqrt{\frac{1}{2}(\sigma_1-\sigma_2)^2 + (\sigma_2-\sigma_3)^2 + (\sigma_3-\sigma_1)^2} - \sigma_y = 0$$

**トレスカ降伏条件：**

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
        """塑性ひずみ増分の計算"""
        pass
```

---

## 6. 有限変形理論

### 6.1 有限変形の課題

変形が大きい場合、より精密な記述が必要である：

- ひずみ計量の選択
- 客観率（ヤウマン、グリーン・ナディ）
- 安定性

### 6.2 構成方程式

```python
class FiniteDeformation:
    """
    有限変形理論
    """
    
    def compute_second_piola_kirchhoff(
        self,
        deformation_gradient: Tensor,
        strain_energy: Callable
    ) -> Tensor:
        """第二P-K応力の計算"""
        # S = ∂W/∂E
        pass
```

---

## 7. 他のスケールとのインターフェース

### 7.1 統計力学 (PS-L1) とのインターフェース

```
統計力学 → 連続体力学：
- 分子運動論から湧現
- 巨視的パラメータの微視的基盤
```

### 7.2 流体力学とのインターフェース

```
連続体力学 → 流体力学：
- 流れは特殊な連続体
- 応力テンソルが圧力に簡略化
```

---

*本文書は連続体力学スケールの物理フレームワークを処理する。*
*連続体力学は工学的科学の基盤であり、応用範囲が広い。*
