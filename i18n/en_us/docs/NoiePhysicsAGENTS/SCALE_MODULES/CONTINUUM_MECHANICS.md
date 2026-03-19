# CONTINUUM_MECHANICS.md

## Continuum Mechanics (PS-L2, PS-L3)

**Scale:** 10⁻³ ~ 10⁷ m  
**Version:** v1.0  
**Status:** Verified

---

## Overview

This document handles the physical framework of **Continuum Mechanics**. According to the physical scale authority hierarchy definition in NoiePhysicsAGENTS.md §1, PS-L2 and PS-L3 represent the human/classical scale and Earth/geological scale respectively, covering physical phenomena from millimeters to thousands of kilometers.

---

## Critical Safety & Truth Protocol

> **CRITICAL SAFETY & TRUTH PROTOCOL:**
> 1. Comply with PT-AX11 (Stationary Action) and PT-AX14 (Momentum Conservation) from AXIOMS.md
> 2. Continuum mechanics is a thoroughly validated engineering science
> 3. Note the validity range of the continuum assumption
> 4. Audit: Record all anomalies to PHYSICS_AUDIT_TRAIL

---

## 1. Continuum Assumption

### 1.1 Basic Assumptions

Continuum mechanics assumes that matter can be treated as continuously distributed points:

```
┌─────────────────────────────────────────────────────────┐
│              Continuum vs. Discrete Particles            │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Discrete view:                                         │
│  ○ ○ ○ ○ ○ ○ ○ ○ ○                                    │
│  ○ ○ ○ ○ ○ ○ ○ ○ ○  ← Atoms/molecules                │
│  ○ ○ ○ ○ ○ ○ ○ ○ ○                                    │
│                                                         │
│  Continuum view:                                        │
│  ═══════════════════════════════                        │
│  ρ(x,y,z) ← Density field                             │
│  v(x,y,z) ← Velocity field                            │
│                                                         │
│  Validity: Characteristic scale >> Intermolecular spacing│
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 1.2 Kinematic Descriptions

**Lagrangian Description** (Material coordinates):

$$\mathbf{x} = \mathbf{x}(\mathbf{X}, t)$$

**Eulerian Description** (Spatial coordinates):

$$\mathbf{v} = \mathbf{v}(\mathbf{x}, t)$$

```python
class KinematicsDescription:
    """
    Kinematic Descriptions
    """
    
    def lagrangian_to_eulerian(
        self,
        field_lagrangian: Field,
        deformation_gradient: Tensor
    ) -> Field:
        """Lagrangian → Eulerian"""
        pass
    
    def compute_deformation_gradient(
        self,
        initial_position: Vector3D,
        current_position: Vector3D
    ) -> Tensor:
        """Compute deformation gradient"""
        # F = ∂x/∂X
        pass
```

---

## 2. Stress and Strain

### 2.1 Strain Tensor

**Green-Lagrange Strain**:

$$E_{ij} = \frac{1}{2}\left(\frac{\partial u_i}{\partial X_j} + \frac{\partial u_j}{\partial X_i} + \frac{\partial u_k}{\partial X_i}\frac{\partial u_k}{\partial X_j}\right)$$

**Almansi Strain** (linear):

$$\varepsilon_{ij} = \frac{1}{2}\left(\frac{\partial u_i}{\partial x_j} + \frac{\partial u_j}{\partial x_i}\right)$$

```python
class StrainTensor:
    """
    Strain Tensor
    """
    
    def compute_linear_strain(
        self,
        displacement_gradient: Tensor
    ) -> Tensor:
        """Compute linear strain"""
        return 0.5 * (displacement_gradient + displacement_gradient.T)
    
    def compute_green_lagrange(
        self,
        displacement_gradient: Tensor
    ) -> Tensor:
        """Compute Green-Lagrange strain"""
        return 0.5 * (displacement_gradient + displacement_gradient.T + 
                      displacement_gradient.T @ displacement_gradient)
```

### 2.2 Stress Tensor

**Cauchy Stress** (true stress):

$$\sigma_{ij} = \lim_{\Delta A_j \to 0} \frac{\Delta F_i}{\Delta A_j}$$

```python
class StressTensor:
    """
    Stress Tensor
    """
    
    def compute_cauchy_stress(
        self,
        force: Vector3D,
        area: Vector3D
    ) -> Tensor:
        """Compute Cauchy stress"""
        return force.outer(area.normalized()) / area.magnitude()
```

### 2.3 Stress-Strain Relationship

**Hooke's Law** (isotropic linear elasticity):

$$\sigma_{ij} = \lambda \varepsilon_{kk}\delta_{ij} + 2\mu\varepsilon_{ij}$$

Where $\lambda$ and $\mu$ are Lamé constants.

```python
class ConstitutiveRelation:
    """
    Constitutive Relation
    """
    
    def hooke_isotropic(
        self,
        strain: Tensor,
        youngs_modulus: float,
        poisson_ratio: float
    ) -> Tensor:
        """Isotropic Hooke's Law"""
        lam = youngs_modulus * poisson_ratio / ((1 + poisson_ratio) * (1 - 2 * poisson_ratio))
        mu = youngs_modulus / (2 * (1 + poisson_ratio))
        
        trace = np.trace(strain)
        return lam * trace * np.eye(3) + 2 * mu * strain
```

---

## 3. Equations of Motion

### 3.1 Equilibrium Equations

**Momentum Balance**:

$$\rho \frac{Dv_i}{Dt} = \frac{\partial \sigma_{ij}}{\partial x_j} + \rho b_i$$

**Mass Conservation** (Continuity equation):

$$\frac{D\rho}{Dt} + \rho \frac{\partial v_i}{\partial x_i} = 0$$

**Energy Conservation**:

$$\rho \frac{De}{Dt} = \sigma_{ij}\frac{\partial v_i}{\partial x_j} + \frac{\partial q_i}{\partial x_i} + \rho r$$

```python
class ConservationLaws:
    """
    Conservation Laws
    """
    
    def momentum_equation(
        self,
        density: float,
        velocity: Vector3D,
        stress: Tensor,
        body_force: Vector3D
    ) -> Vector3D:
        """Momentum equation"""
        divergence_stress = stress.divergence()
        return density * velocity.time_derivative() - divergence_stress - density * body_force
    
    def continuity_equation(
        self,
        density: float,
        velocity: Vector3D
    ) -> float:
        """Continuity equation"""
        return density.time_derivative() + density * velocity.divergence()
```

---

## 4. Elasticity

### 4.1 Boundary Value Problems

```python
class ElasticBoundaryValueProblem:
    """
    Elastic Boundary Value Problem
    """
    
    def solve_displacement(
        self,
        domain: Domain,
        boundary_conditions: BoundaryConditions,
        material_properties: MaterialProperties
    ) -> DisplacementField:
        """
        Solve displacement field
        
        Methods: Finite Element, Boundary Element, Analytical solutions
        """
        pass
```

### 4.2 Boundary Conditions

| Type | Mathematical Expression | Physical Meaning |
|------|------------------------|-----------------|
| Displacement boundary | u = ū | Fixed support |
| Force boundary | σ·n = t̄ | Load |
| Mixed boundary | Combination | Elastic support |

---

## 5. Plasticity

### 5.1 Yield Criteria

**von Mises Yield Criterion**:

$$f(\sigma) = \sqrt{\frac{1}{2}(\sigma_1-\sigma_2)^2 + (\sigma_2-\sigma_3)^2 + (\sigma_3-\sigma_1)^2} - \sigma_y = 0$$

**Tresca Yield Criterion**:

$$f(\sigma) = \max(|\sigma_1-\sigma_2|, |\sigma_2-\sigma_3|, |\sigma_3-\sigma_1|) - \sigma_y = 0$$

### 5.2 Flow Theory

```python
class PlasticFlow:
    """
    Plastic Flow
    """
    
    def compute_plastic_strain_increment(
        self,
        stress: Tensor,
        yield_function: YieldFunction,
        hardening_law: HardeningLaw
    ) -> Tensor:
        """Compute plastic strain increment"""
        pass
```

---

## 6. Finite Deformation Theory

### 6.1 Challenges of Finite Deformation

When deformations are large, more accurate descriptions are needed:

- Strain measure selection
- Objective rates (Jaumann, Green-Naghdi)
- Stability

### 6.2 Constitutive Equations

```python
class FiniteDeformation:
    """
    Finite Deformation Theory
    """
    
    def compute_second_piola_kirchhoff(
        self,
        deformation_gradient: Tensor,
        strain_energy: Callable
    ) -> Tensor:
        """Compute second P-K stress"""
        # S = ∂W/∂E
        pass
```

---

## 7. Interfaces with Other Scales

### 7.1 Interface with Statistical Mechanics (PS-L1)

```
Statistical Mechanics → Continuum Mechanics:
|- Emergence from molecular kinetics
|- Microscopic foundation of macroscopic parameters
```

### 7.2 Interface with Fluid Dynamics

```
Continuum Mechanics → Fluid Dynamics:
|- Flow is a special case of continuum
|- Stress tensor simplifies to pressure
```

---

*This document handles the physical framework at the continuum mechanics scale.*
*Continuum mechanics is the foundation of engineering science with wide applications.*
