# COLLISION_SYSTEM.md

## L3 - 碰撞檢測與響應

> **WARNING:** 本模組是 DYNAMICS_ENGINE 的碰撞系統子模組。
> **版本:** v1.1

---

## 概述

本文檔定義 NoiePhysicsAGENTS 的碰撞檢測與響應系統，處理實體之間的碰撞幾何和動力學。系統支援從簡單的球體碰撞到複雜的多體碰撞場景，涵蓋彈性、塑性、黏彈性等多種材料響應模型。

---

## 1. 碰撞幾何檢測演算法

### 1.1 分離軸定理 (Separating Axis Theorem, SAT)

分離軸定理是檢測凸多面體碰撞的核心演算法。兩個凸多面體不相交若且唯若存在一個軸，使得兩個物體在該軸上的投影區間不重疊。

```python
import numpy as np
from typing import List, Tuple, Optional

class SATCollisionDetector:
    """
    分離軸定理碰撞檢測器
    """
    
    def __init__(self, tolerance: float = 1e-6):
        self.tolerance = tolerance
    
    def get_separating_axis(
        self,
        polyhedron_a: 'Polyhedron',
        polyhedron_b: 'Polyhedron'
    ) -> Optional[np.ndarray]:
        """
        尋找分離軸
        若存在分離軸則返回該軸，否則返回 None（表示碰撞）
        """
        candidate_axes = []
        
        candidate_axes.extend(self._get_face_normals(polyhedron_a))
        candidate_axes.extend(self._get_face_normals(polyhedron_b))
        
        candidate_axes.extend(self._get_edge_cross_products(polyhedron_a, polyhedron_b))
        
        for axis in candidate_axes:
            if axis is None or np.linalg.norm(axis) < self.tolerance:
                continue
                
            axis = axis / np.linalg.norm(axis)
            
            proj_a = self._project_polyhedron(polyhedron_a, axis)
            proj_b = self._project_polyhedron(polyhedron_b, axis)
            
            if not self._overlap(proj_a, proj_b):
                return axis
        
        return None
    
    def _get_face_normals(self, polyhedron: 'Polyhedron') -> List[np.ndarray]:
        """取得多面體所有面的法向量"""
        normals = []
        for face in polyhedron.faces:
            normal = np.cross(face.edge1, face.edge2)
            if np.linalg.norm(normal) > self.tolerance:
                normals.append(normal)
        return normals
    
    def _get_edge_cross_products(
        self,
        polyhedron_a: 'Polyhedron',
        polyhedron_b: 'Polyhedron'
    ) -> List[np.ndarray]:
        """取得兩多面體邊緣的叉積候選軸"""
        cross_products = []
        
        for edge_a in polyhedron_a.edges:
            for edge_b in polyhedron_b.edges:
                cross = np.cross(edge_a.direction, edge_b.direction)
                cross_products.append(cross)
        
        return cross_products
    
    def _project_polyhedron(
        self,
        polyhedron: 'Polyhedron',
        axis: np.ndarray
    ) -> Tuple[float, float]:
        """將多面體投影到軸上，返回 [min, max]"""
        vertices = polyhedron.vertices
        
        projections = [np.dot(v, axis) for v in vertices]
        
        return (min(projections), max(projections))
    
    def _overlap(self, interval_a: Tuple[float, float], interval_b: Tuple[float, float]) -> bool:
        """檢測兩個區間是否重疊"""
        return not (interval_a[1] < interval_b[0] or interval_b[1] < interval_a[0])
    
    def check_collision(
        self,
        polyhedron_a: 'Polyhedron',
        polyhedron_b: 'Polyhedron'
    ) -> bool:
        """檢測兩個凸多面體是否碰撞"""
        separating_axis = self.get_separating_axis(polyhedron_a, polyhedron_b)
        return separating_axis is None
```

### 1.2 包容球體層次結構 (Bounding Sphere Hierarchy, BVH)

對於複雜場景，使用包容球體層次結構可以大幅減少碰撞檢測計算量。

```python
class BoundingSphere:
    """
    包容球體
    """
    
    def __init__(self, center: np.ndarray, radius: float):
        self.center = center
        self.radius = radius
    
    def contains_point(self, point: np.ndarray) -> bool:
        """檢查點是否在球體內"""
        return np.linalg.norm(point - self.center) <= self.radius
    
    def intersects(self, other: 'BoundingSphere') -> bool:
        """檢查兩個包容球體是否相交"""
        distance = np.linalg.norm(self.center - other.center)
        return distance < (self.radius + other.radius)
    
    def merge(self, other: 'BoundingSphere') -> 'BoundingSphere':
        """合併兩個包容球體，產生包含兩者的最小球體"""
        diff = other.center - self.center
        distance = np.linalg.norm(diff)
        
        if distance <= abs(self.radius - other.radius):
            if self.radius >= other.radius:
                return BoundingSphere(self.center.copy(), self.radius)
            else:
                return BoundingSphere(other.center.copy(), other.radius)
        
        new_radius = (self.radius + distance + other.radius) / 2
        new_center = self.center + diff * ((new_radius - self.radius) / distance)
        
        return BoundingSphere(new_center, new_radius)


class BVHNode:
    """
    BVH 節點
    """
    
    def __init__(
        self,
        bounding_sphere: BoundingSphere,
        left: Optional['BVHNode'] = None,
        right: Optional['BVHNode'] = None,
        objects: Optional[List] = None
    ):
        self.bounding_sphere = bounding_sphere
        self.left = left
        self.right = right
        self.objects = objects if objects is not None else []


class BoundingSphereHierarchy:
    """
    包容球體層次結構
    """
    
    def __init__(self):
        self.root: Optional[BVHNode] = None
        self.objects: List = []
    
    def build(self, objects: List, max_objects_per_leaf: int = 4):
        """構建 BVH"""
        self.objects = objects
        self.root = self._build_recursive(objects, max_objects_per_leaf)
    
    def _build_recursive(
        self,
        objects: List,
        max_objects_per_leaf: int
    ) -> BVHNode:
        if len(objects) <= max_objects_per_leaf:
            center = np.mean([obj.position for obj in objects], axis=0)
            max_dist = max(np.linalg.norm(obj.position - center) for obj in objects)
            bounding = BoundingSphere(center, max_dist)
            return BVHNode(bounding, objects=objects)
        
        center = np.mean([obj.position for obj in objects], axis=0)
        
        sorted_objects = sorted(objects, key=lambda o: np.linalg.norm(o.position - center))
        
        mid = len(sorted_objects) // 2
        left_objects = sorted_objects[:mid]
        right_objects = sorted_objects[mid:]
        
        left_node = self._build_recursive(left_objects, max_objects_per_leaf)
        right_node = self._build_recursive(right_objects, max_objects_per_leaf)
        
        merged_bounds = left_node.bounding_sphere.merge(right_node.bounding_sphere)
        
        return BVHNode(merged_bounds, left=left_node, right=right_node)
    
    def query(self, query_sphere: BoundingSphere) -> List:
        """查詢與查詢球體相交的所有物件"""
        results = []
        self._query_recursive(self.root, query_sphere, results)
        return results
    
    def _query_recursive(
        self,
        node: BVHNode,
        query_sphere: BoundingSphere,
        results: List
    ):
        if node is None:
            return
        
        if not node.bounding_sphere.intersects(query_sphere):
            return
        
        if node.objects:
            results.extend(node.objects)
            return
        
        self._query_recursive(node.left, query_sphere, results)
        self._query_recursive(node.right, query_sphere, results)
```

---

## 2. 接觸力學模型

### 2.1 彈性接觸模型

基於 Hertz 理論的彈性接觸模型，假設材料為均匀、各向同性：

```python
class ElasticContactModel:
    """
    彈性接觸模型 (Hertzian Contact)
    """
    
    def __init__(self, youngs_modulus: float, poisson_ratio: float):
        """
        初始化彈性模型
        
        參數:
            youngs_modulus: 楊氏模量 E (Pa)
            poisson_ratio: 泊松比 ν
        """
        self.E = youngs_modulus
        self.nu = poisson_ratio
        
        self.effective_modulus = self._compute_effective_modulus()
    
    def _compute_effective_modulus(self) -> float:
        """計算有效彈性模量"""
        return self.E / (2 * (1 + self.nu))
    
    def compute_contact_radius(
        self,
        radius_a: float,
        radius_b: float,
        normal_force: float
    ) -> float:
        """
        計算接觸半徑
        
        公式: a = [3FR* / (4E*)]^(1/3)
        其中 R* = (1/R_a + 1/R_b)^(-1) 為有效曲率半徑
        """
        effective_radius = 1.0 / (1.0 / radius_a + 1.0 / radius_b)
        contact_radius = (3 * normal_force * effective_radius / self.effective_modulus) ** (1.0 / 3.0)
        return contact_radius
    
    def compute_penetration(
        self,
        radius_a: float,
        radius_b: float,
        normal_force: float
    ) -> float:
        """
        計算穿透深度 (approach)
        
        公式: δ = a^2 / R* = [3FR* / (4E*^2)]^(2/3)
        """
        effective_radius = 1.0 / (1.0 / radius_a + 1.0 / radius_b)
        contact_radius = self.compute_contact_radius(radius_a, radius_b, normal_force)
        return contact_radius ** 2 / effective_radius
    
    def compute_contact_force(
        self,
        radius_a: float,
        radius_b: float,
        penetration: float
    ) -> float:
        """
        根據穿透深度計算接觸力
        
        公式: F = (4/3)E* R*^(1/2) δ^(3/2)
        """
        effective_radius = 1.0 / (1.0 / radius_a + 1.0 / radius_b)
        force = (4.0 / 3.0) * self.effective_modulus * (effective_radius ** 0.5) * (penetration ** 1.5)
        return force
```

### 2.2 塑性接觸模型

當接觸壓力超過屈服應力時，材料發生塑性變形：

```python
class PlasticContactModel:
    """
    塑性接觸模型
    """
    
    def __init__(
        self,
        yield_strength: float,
        hardening_modulus: float,
        elastic_modulus: float
    ):
        self.yield_strength = yield_strength
        self.H = hardening_modulus
        self.E = elastic_modulus
    
    def compute_plastic_contact_radius(
        self,
        radius_a: float,
        radius_b: float,
        normal_force: float
    ) -> float:
        """計算塑性接觸半徑"""
        effective_radius = 1.0 / (1.0 / radius_a + 1.0 / radius_b)
        
        if normal_force <= self.yield_strength * np.pi * effective_radius ** 2:
            return self._compute_elastic_contact_radius(radius_a, radius_b, normal_force)
        
        contact_radius = np.sqrt(normal_force / (np.pi * self.yield_strength))
        return contact_radius
    
    def _compute_elastic_contact_radius(
        self,
        radius_a: float,
        radius_b: float,
        normal_force: float
    ) -> float:
        """計算彈性接觸半徑"""
        effective_radius = 1.0 / (1.0 / radius_a + 1.0 / radius_b)
        return (3 * normal_force * effective_radius / (4 * self.E)) ** (1.0 / 3.0)
```

### 2.3 黏彈性接觸模型

黏彈性材料表現出時間依賴的應力-應變關係：

```python
class ViscoelasticContactModel:
    """
    黏彈性接觸模型
    """
    
    def __init__(
        self,
        elastic_modulus: float,
        viscosity: float,
        relaxation_time: float
    ):
        self.E = elastic_modulus
        self.eta = viscosity
        self.tau = relaxation_time
    
    def compute_relaxed_modulus(self) -> float:
        """計算弛豫模量"""
        return self.E * self.tau / (self.eta + self.tau)
    
    def compute_contact_force(
        self,
        radius_a: float,
        radius_b: float,
        penetration: float,
        time: float,
        loading_rate: float
    ) -> float:
        """
        計算黏彈性接觸力
        
        使用 Kelvin-Voigt 模型
        """
        effective_radius = 1.0 / (1.0 / radius_a + 1.0 / radius_b)
        
        elastic_force = (4.0 / 3.0) * self.E * (effective_radius ** 0.5) * (penetration ** 1.5)
        
        damping_force = (4.0 / 3.0) * self.eta * (effective_radius ** 0.5) * (loading_rate) * (penetration ** 0.5)
        
        relaxation_factor = 1 - np.exp(-time / self.tau)
        
        return elastic_force * relaxation_factor + damping_force
```

---

## 3. 摩擦模型

### 3.1 庫侖摩擦

庫侖摩擦是描述乾摩擦的基本模型：

$$F_f = \mu N$$

其中 $\mu$ 為摩擦係數，$N$ 為法向力。

```python
class CoulombFriction:
    """
    庫侖摩擦模型
    """
    
    def __init__(
        self,
        static_friction_coefficient: float,
        dynamic_friction_coefficient: float,
        transition_velocity: float = 0.01
    ):
        self.mu_s = static_friction_coefficient
        self.mu_d = dynamic_friction_coefficient
        self.v_transition = transition_velocity
    
    def compute_friction_force(
        self,
        normal_force: float,
        tangential_velocity: np.ndarray,
        contact_point: np.ndarray
    ) -> np.ndarray:
        """
        計算摩擦力向量
        
        參數:
            normal_force: 法向力大小
            tangential_velocity: 切向速度向量
            contact_point: 接觸點位置
        
        返回:
            摩擦力向量
        """
        speed = np.linalg.norm(tangential_velocity)
        
        if speed < self.v_transition:
            mu_effective = self.mu_s
        else:
            mu_effective = self._interpolate_friction(speed)
        
        max_friction = mu_effective * normal_force
        
        if speed < 1e-6:
            return np.zeros(3)
        
        friction_direction = -tangential_velocity / speed
        friction_magnitude = min(max_friction, mu_effective * normal_force)
        
        return friction_direction * friction_magnitude
    
    def _interpolate_friction(self, speed: float) -> float:
        """在靜摩擦和動摩擦之間平滑過渡"""
        t = min(1.0, (speed - self.v_transition) / (self.v_transition * 10))
        return self.mu_s + t * (self.mu_d - self.mu_s)
```

### 3.2 滾動摩擦

滾動摩擦發生在旋轉物體與表面接觸時：

```python
class RollingFriction:
    """
    滾動摩擦模型
    """
    
    def __init__(self, rolling_resistance_coefficient: float):
        self.C_rr = rolling_resistance_coefficient
    
    def compute_rolling_torque(
        self,
        normal_force: float,
        wheel_radius: float,
        angular_velocity: np.ndarray,
        contact_normal: np.ndarray
    ) -> np.ndarray:
        """
        計算滾動摩擦力矩
        
        公式: M_roll = C_rr * N * r * û
        其中 û 是接觸法線方向的單位向量
        """
        rolling_resistance_force = self.C_rr * normal_force
        
        torque_direction = np.cross(contact_normal, angular_velocity)
        if np.linalg.norm(torque_direction) > 1e-6:
            torque_direction = torque_direction / np.linalg.norm(torque_direction)
        
        torque_magnitude = rolling_resistance_force * wheel_radius
        
        return torque_direction * torque_magnitude
    
    def compute_deceleration(
        self,
        velocity: np.ndarray,
        angular_velocity: np.ndarray,
        normal_force: float,
        wheel_radius: float
    ) -> tuple:
        """
        計算滾動導致的減速度和角加速度
        """
        linear_decel = -self.C_rr * normal_force * np.sign(velocity)
        
        rolling_torque = self.compute_rolling_torque(
            normal_force, wheel_radius, angular_velocity, np.array([0, 1, 0])
        )
        
        return linear_decel, rolling_torque
```

---

## 4. 碰撞回應策略

### 4.1 脈衝-Based 碰撞回應

基於動量守恆的脈衝方法：

```python
class ImpulseBasedCollisionResponse:
    """
    脈衝-Based 碰撞回應
    """
    
    def __init__(self, restitution: float = 0.8):
        self.restitution = restitution
    
    def compute_collision_impulse(
        self,
        pos_a: np.ndarray,
        pos_b: np.ndarray,
        vel_a: np.ndarray,
        vel_b: np.ndarray,
        mass_a: float,
        mass_b: float,
        normal: np.ndarray,
        restitution: Optional[float] = None
    ) -> np.ndarray:
        """
        計算碰撞脈衝
        
        公式: j = -(1 + e) * v_rel · n / (1/m_a + 1/m_b)
        
        參數:
            pos_a, pos_b: 碰撞點位置
            vel_a, vel_b: 碰撞前速度
            mass_a, mass_b: 質量
            normal: 接觸法向量（從 A 指向 B）
            restitution: 恢復係數
        
        返回:
            脈衝向量（作用於 A），B 受到相反方向的脈衝
        """
        e = restitution if restitution is not None else self.restitution
        
        relative_velocity = vel_a - vel_b
        
        vel_along_normal = np.dot(relative_velocity, normal)
        
        if vel_along_normal > 0:
            return np.zeros(3)
        
        inv_mass_a = 1.0 / mass_a
        inv_mass_b = 1.0 / mass_b
        inv_mass_sum = inv_mass_a + inv_mass_b
        
        j = -(1 + e) * vel_along_normal / inv_mass_sum
        
        return j * normal
    
    def apply_impulse(
        self,
        body_a: 'RigidBody',
        body_b: 'RigidBody',
        impulse: np.ndarray,
        contact_point: np.ndarray
    ):
        """
        將脈衝應用到剛體
        """
        body_a.velocity += impulse / body_a.mass
        
        r_a = contact_point - body_a.position
        body_a.angular_velocity += np.cross(r_a, impulse) / body_a.inertia
        
        body_b.velocity -= impulse / body_b.mass
        
        r_b = contact_point - body_b.position
        body_b.angular_velocity -= np.cross(r_b, impulse) / body_b.inertia
```

### 4.2 Penalty-Based 碰撞回應

使用彈簧-阻尼模型進行連續碰撞回應：

```python
class PenaltyBasedCollisionResponse:
    """
    Penalty-Based 碰撞回應（彈簧-阻尼模型）
    """
    
    def __init__(
        self,
        stiffness: float = 1e6,
        damping: float = 1000.0,
        penetration_threshold: float = 0.001
    ):
        self.stiffness = stiffness
        self.damping = damping
        self.penetration_threshold = penetration_threshold
    
    def compute_contact_force(
        self,
        penetration: float,
        penetration_velocity: float,
        normal: np.ndarray,
        mass_effective: float
    ) -> np.ndarray:
        """
        計算 penalty 接觸力
        
        公式: F = k * δ + c * δ̇
        其中 k 為剛度，c 為阻尼係數
        """
        if penetration < self.penetration_threshold:
            return np.zeros(3)
        
        spring_force = self.stiffness * penetration
        
        damping_force = self.damping * penetration_velocity
        
        total_force_magnitude = spring_force + damping_force
        
        return -normal * total_force_magnitude
    
    def compute_impulse_from_force(
        self,
        contact_force: np.ndarray,
        dt: float
    ) -> np.ndarray:
        """從連續力計算等效脈衝"""
        return contact_force * dt
```

### 4.3 約束-Based 碰撞回應

使用約束求解器處理多體碰撞：

```python
class ConstraintBasedCollisionResponse:
    """
    約束-Based 碰撞回應
    """
    
    def __init__(self, tolerance: float = 1e-5, max_iterations: int = 100):
        self.tolerance = tolerance
        self.max_iterations = max_iterations
    
    def create_contact_constraint(
        self,
        body_a: 'RigidBody',
        body_b: 'RigidBody',
        contact_point: np.ndarray,
        normal: np.ndarray
    ) -> 'ContactConstraint':
        """
        創建接觸約束
        """
        return ContactConstraint(
            body_a=body_a,
            body_b=body_b,
            contact_point=contact_point,
            normal=normal,
            restitution=self.restitution
        )
    
    def solve_constraints(
        self,
        bodies: List['RigidBody'],
        constraints: List['ContactConstraint']
    ) -> List[np.ndarray]:
        """
        求解所有接觸約束
        
        使用 Gauss-Seidel 迭代求解
        """
        impulses = [np.zeros(3) for _ in constraints]
        
        for iteration in range(self.max_iterations):
            max_correction = 0.0
            
            for i, constraint in enumerate(constraints):
                correction = self._solve_single_constraint(
                    constraint, bodies, impulses[i]
                )
                max_correction = max(max_correction, np.linalg.norm(correction))
                impulses[i] += correction
            
            if max_correction < self.tolerance:
                break
        
        return impulses
    
    def _solve_single_constraint(
        self,
        constraint: 'ContactConstraint',
        bodies: List['RigidBody'],
        current_impulse: np.ndarray
    ) -> np.ndarray:
        """求解單個約束"""
        body_a = constraint.body_a
        body_b = constraint.body_b
        
        r_a = constraint.contact_point - body_a.position
        r_b = constraint.contact_point - body_b.position
        
        v_rel = body_a.velocity + np.cross(body_a.angular_velocity, r_a) - \
                body_b.velocity - np.cross(body_b.angular_velocity, r_b)
        
        vel_along_normal = np.dot(v_rel, constraint.normal)
        
        if vel_along_normal > 0:
            return np.zeros(3)
        
        inv_mass_a = 1.0 / body_a.mass
        inv_mass_b = 1.0 / body_b.mass
        
        r_a_cross_n = np.cross(r_a, constraint.normal)
        r_b_cross_n = np.cross(r_b, constraint.normal)
        
        angular_term = np.dot(
            r_a_cross_n,
            r_a_cross_n / body_a.inertia
        ) + np.dot(
            r_b_cross_n,
            r_b_cross_n / body_b.inertia
        )
        
        denom = inv_mass_a + inv_mass_b + angular_term
        
        if abs(denom) < 1e-10:
            return np.zeros(3)
        
        j = -(1 + constraint.restitution) * vel_along_normal / denom
        
        impulse = j * constraint.normal
        
        body_a.velocity += impulse * inv_mass_a
        body_a.angular_velocity += np.cross(r_a, impulse) / body_a.inertia
        
        body_b.velocity -= impulse * inv_mass_b
        body_b.angular_velocity -= np.cross(r_b, impulse) / body_b.inertia
        
        return impulse


class ContactConstraint:
    """接觸約束"""
    
    def __init__(
        self,
        body_a: 'RigidBody',
        body_b: 'RigidBody',
        contact_point: np.ndarray,
        normal: np.ndarray,
        restitution: float = 0.8
    ):
        self.body_a = body_a
        self.body_b = body_b
        self.contact_point = contact_point
        self.normal = normal / np.linalg.norm(normal)
        self.restitution = restitution
```

---

## 5. 高速碰撞與穿隧效應處理

### 5.1 連續碰撞檢測 (Continuous Collision Detection, CCD)

當物體速度較高時，離散碰撞檢測可能漏掉碰撞（穿隧效應）。CCD 確保在時間間隔內所有碰撞都被檢測到：

```python
class ContinuousCollisionDetection:
    """
    連續碰撞檢測
    """
    
    def __init__(self, max_substeps: int = 8):
        self.max_substeps = max_substeps
    
    def compute_time_of_impact(
        self,
        pos_a_start: np.ndarray,
        pos_a_end: np.ndarray,
        vel_a: np.ndarray,
        radius_a: float,
        pos_b: np.ndarray,
        radius_b: float
    ) -> Optional[float]:
        """
        計算碰撞時間
        
        參數:
            pos_a_start: A 起始位置
            pos_a_end: A 結束位置
            vel_a: A 的速度
            radius_a: A 的半徑
            pos_b: B 的位置（假設靜止）
            radius_b: B 的半徑
        
        返回:
            碰撞時間（0 到 1 之间），若無碰撞則返回 None
        """
        relative_pos = pos_a_start - pos_b
        relative_vel = vel_a
        
        a = np.dot(relative_vel, relative_vel)
        b = 2 * np.dot(relative_pos, relative_vel)
        c = np.dot(relative_pos, relative_pos) - (radius_a + radius_b) ** 2
        
        discriminant = b * b - 4 * a * c
        
        if discriminant < 0:
            return None
        
        sqrt_disc = np.sqrt(discriminant)
        
        t1 = (-b - sqrt_disc) / (2 * a)
        t2 = (-b + sqrt_disc) / (2 * a)
        
        if 0 <= t1 <= 1:
            return t1
        if 0 <= t2 <= 1:
            return t2
        
        return None
    
    def sweep_test(
        self,
        body: 'RigidBody',
        obstacles: List['RigidBody'],
        dt: float
    ) -> List[CollisionEvent]:
        """
        掃描測試：檢測運動路徑上的所有碰撞
        """
        events = []
        
        pos_start = body.position
        vel = body.velocity
        pos_end = pos_start + vel * dt
        
        for obstacle in obstacles:
            tof = self.compute_time_of_impact(
                pos_start, pos_end, vel,
                body.collision_radius,
                obstacle.position,
                obstacle.collision_radius
            )
            
            if tof is not None:
                contact_point = pos_start + vel * tof * dt
                events.append(CollisionEvent(
                    body_a=body,
                    body_b=obstacle,
                    time_of_impact=tof * dt,
                    contact_point=contact_point
                ))
        
        return events
```

### 5.2 子步驟細分

將時間步細分為多個子步驟以提高碰撞檢測精度：

```python
class SubstepCollisionSolver:
    """
    子步驟碰撞求解器
    """
    
    def __init__(self, num_substeps: int = 4):
        self.num_substeps = num_substeps
    
    def solve_with_substeps(
        self,
        bodies: List['RigidBody'],
        dt: float,
        collision_system: 'CollisionSystem'
    ) -> None:
        """
        使用子步驟求解碰撞
        """
        sub_dt = dt / self.num_substeps
        
        for step in range(self.num_substeps):
            for body in bodies:
                body.position += body.velocity * sub_dt
                body.orientation += body.angular_velocity * sub_dt
            
            contacts = collision_system.detect_collisions(bodies)
            
            for contact in contacts:
                collision_system.resolve_collision(contact)
            
            for body in bodies:
                body.apply_gravity(sub_dt)
                body.apply_damping(sub_dt)
```

---

## 6. 多體碰撞系統

### 6.1 衝突圖與衝突矩陣

```python
import numpy as np
from collections import defaultdict

class CollisionGraph:
    """
    衝突圖表示多體碰撞關係
    """
    
    def __init__(self):
        self.nodes = set()
        self.edges = defaultdict(set)
    
    def add_collision(self, body_a, body_b):
        """添加碰撞邊"""
        self.nodes.add(body_a)
        self.nodes.add(body_b)
        self.edges[body_a].add(body_b)
        self.edges[body_b].add(body_a)
    
    def get_collision_groups(self) -> List[set]:
        """獲取連通的碰撞組"""
        visited = set()
        groups = []
        
        def dfs(node, group):
            visited.add(node)
            group.add(node)
            for neighbor in self.edges[node]:
                if neighbor not in visited:
                    dfs(neighbor, group)
        
        for node in self.nodes:
            if node not in visited:
                group = set()
                dfs(node, group)
                groups.append(group)
        
        return groups


class MultiBodyCollisionSystem:
    """
    多體碰撞系統
    """
    
    def __init__(
        self,
        collision_detector: 'SATCollisionDetector',
        response_strategy: 'ImpulseBasedCollisionResponse'
    ):
        self.detector = collision_detector
        self.response = response_strategy
        self.collision_graph = CollisionGraph()
    
    def detect_and_resolve(
        self,
        bodies: List['RigidBody'],
        dt: float
    ) -> List['CollisionContact']:
        """
        檢測並解決所有碰撞
        """
        self.collision_graph = CollisionGraph()
        
        contacts = self._broad_phase(bodies)
        
        contacts = self._narrow_phase(contacts)
        
        contacts = self._filter_redundant_contacts(contacts)
        
        for contact in contacts:
            self._resolve_contact(contact)
        
        return contacts
    
    def _broad_phase(self, bodies: List['RigidBody']) -> List[tuple]:
        """粗測階段：使用 BVH 過濾不可能碰撞的物體"""
        bvh = BoundingSphereHierarchy()
        bvh.build(bodies)
        
        potential_contacts = []
        
        for i, body_a in enumerate(bodies):
            query_sphere = BoundingSphere(
                body_a.position,
                body_a.collision_radius * 2
            )
            candidates = bvh.query(query_sphere)
            
            for body_b in candidates:
                if body_a.id < body_b.id:
                    potential_contacts.append((body_a, body_b))
        
        return potential_contacts
    
    def _narrow_phase(
        self,
        potential_contacts: List[tuple]
    ) -> List['CollisionContact']:
        """細測階段：精確碰撞檢測"""
        contacts = []
        
        for body_a, body_b in potential_contacts:
            if self.detector.check_collision(body_a.polyhedron, body_b.polyhedron):
                normal, penetration = self._compute_contact_info(body_a, body_b)
                contact = CollisionContact(
                    body_a=body_a,
                    body_b=body_b,
                    normal=normal,
                    penetration=penetration
                )
                contacts.append(contact)
                
                self.collision_graph.add_collision(body_a, body_b)
        
        return contacts
    
    def _compute_contact_info(
        self,
        body_a: 'RigidBody',
        body_b: 'RigidBody'
    ) -> tuple:
        """計算接觸點信息"""
        direction = body_b.position - body_a.position
        distance = np.linalg.norm(direction)
        
        if distance < 1e-6:
            normal = np.array([1, 0, 0])
        else:
            normal = direction / distance
        
        penetration = (body_a.collision_radius + body_b.collision_radius) - distance
        
        return normal, max(0, penetration)
    
    def _filter_redundant_contacts(
        self,
        contacts: List['CollisionContact']
    ) -> List['CollisionContact']:
        """過濾冗餘接觸"""
        filtered = []
        
        contact_points = set()
        
        for contact in contacts:
            point_key = (
                tuple(contact.body_a.id),
                tuple(contact.body_b.id)
            )
            
            if point_key not in contact_points:
                filtered.append(contact)
                contact_points.add(point_key)
        
        return filtered
    
    def _resolve_contact(self, contact: 'CollisionContact'):
        """解決單個接觸"""
        impulse = self.response.compute_collision_impulse(
            contact.body_a.position,
            contact.body_b.position,
            contact.body_a.velocity,
            contact.body_b.velocity,
            contact.body_a.mass,
            contact.body_b.mass,
            contact.normal
        )
        
        self.response.apply_impulse(
            contact.body_a,
            contact.body_b,
            impulse,
            contact.body_a.position
        )


class CollisionContact:
    """碰撞接觸"""
    
    def __init__(
        self,
        body_a: 'RigidBody',
        body_b: 'RigidBody',
        normal: np.ndarray,
        penetration: float
    ):
        self.body_a = body_a
        self.body_b = body_b
        self.normal = normal
        self.penetration = penetration
        self.contact_point = (body_a.position + body_b.position) / 2
```

---

*本文檔是 DYNAMICS_ENGINE 的碰撞系統子模組。*
*版本: v1.1*
