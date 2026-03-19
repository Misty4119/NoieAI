# COLLISION_SYSTEM.md

## L3 - 衝突検出と応答

> **WARNING:** このモジュールは DYNAMICS_ENGINE の衝突システムサブモジュールです。
> **バージョン:** v1.1

---

## 概要

このドキュメントは NoiePhysicsAGENTS の衝突検出と応答システムを定義し、実体間の衝突幾何と動力学を処理します。システムは単純な球体衝突から複雑な多体衝突シナリオまで対応し、弾性、塑性、粘弾性などの 다양한 材料応答モデルをサポートします。

---

## 1. 衝突幾何検出アルゴリズム

### 1.1 分離軸定理 (Separating Axis Theorem, SAT)

分離軸定理は凸多面体衝突検出のコアアルゴリズムです。2つの凸多面体が不相交であるのは、ある軸が存在して2つの物体がその軸への射影区間が重ならない場合のみです。

```python
import numpy as np
from typing import List, Tuple, Optional

class SATCollisionDetector:
    """
    分離軸定理衝突検出器
    """
    
    def __init__(self, tolerance: float = 1e-6):
        self.tolerance = tolerance
    
    def get_separating_axis(
        self,
        polyhedron_a: 'Polyhedron',
        polyhedron_b: 'Polyhedron'
    ) -> Optional[np.ndarray]:
        """
        分離軸を探す
        分離軸が存在する場合はその軸を返し、存在しない場合は None を返す（衝突あり）
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
        """多面体のすべての面の法線ベクトルを取得"""
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
        """2つの多面体の辺のクロス積候補軸を取得"""
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
        """多面体を軸に射影し、[min, max] を返す"""
        vertices = polyhedron.vertices
        
        projections = [np.dot(v, axis) for v in vertices]
        
        return (min(projections), max(projections))
    
    def _overlap(self, interval_a: Tuple[float, float], interval_b: Tuple[float, float]) -> bool:
        """2つの区間が重なるかどうか検査"""
        return not (interval_a[1] < interval_b[0] or interval_b[1] < interval_a[0])
    
    def check_collision(
        self,
        polyhedron_a: 'Polyhedron',
        polyhedron_b: 'Polyhedron'
    ) -> bool:
        """2つの凸多面体が衝突するかどうか検査"""
        separating_axis = self.get_separating_axis(polyhedron_a, polyhedron_b)
        return separating_axis is None
```

### 1.2 境界球体階層構造 (Bounding Volume Hierarchy, BVH)

複雑なシナリオでは、境界球体階層構造を使用することで衝突検出計算量を大幅に削減できます。

```python
class BoundingSphere:
    """
    境界球体
    """
    
    def __init__(self, center: np.ndarray, radius: float):
        self.center = center
        self.radius = radius
    
    def contains_point(self, point: np.ndarray) -> bool:
        """点が球体内にあるかどうか検査"""
        return np.linalg.norm(point - self.center) <= self.radius
    
    def intersects(self, other: 'BoundingSphere') -> bool:
        """2つの境界球体が相交差するかどうか検査"""
        distance = np.linalg.norm(self.center - other.center)
        return distance < (self.radius + other.radius)
    
    def merge(self, other: 'BoundingSphere') -> 'BoundingSphere':
        """2つの境界球体をマージし、両方を含む最小の球体を生成"""
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
    BVH ノード
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
    境界球体階層構造
    """
    
    def __init__(self):
        self.root: Optional[BVHNode] = None
        self.objects: List = []
    
    def build(self, objects: List, max_objects_per_leaf: int = 4):
        """BVH を構築"""
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
        """クエリ球体と相交差するすべてのオブジェクトをクエリ"""
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

## 2. 接触力学モデル

### 2.1 弾性接触モデル

Hertz 理論に基づく弾性接触モデルでは、材料が均質で等方性であると仮定します：

```python
class ElasticContactModel:
    """
    弾性接触モデル (Hertzian Contact)
    """
    
    def __init__(self, youngs_modulus: float, poisson_ratio: float):
        """
        弾性モデルを初期化
        
        パラメータ:
            youngs_modulus: ヤング率 E (Pa)
            poisson_ratio: ポアソン比 ν
        """
        self.E = youngs_modulus
        self.nu = poisson_ratio
        
        self.effective_modulus = self._compute_effective_modulus()
    
    def _compute_effective_modulus(self) -> float:
        """有効弾性率を計算"""
        return self.E / (2 * (1 + self.nu))
    
    def compute_contact_radius(
        self,
        radius_a: float,
        radius_b: float,
        normal_force: float
    ) -> float:
        """
        接触半径を計算
        
        公式: a = [3FR* / (4E*)]^(1/3)
        ただし R* = (1/R_a + 1/R_b)^(-1) は有効曲率半径
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
        穿透深度を計算 (approach)
        
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
        穿透深度から接触力を計算
        
        公式: F = (4/3)E* R*^(1/2) δ^(3/2)
        """
        effective_radius = 1.0 / (1.0 / radius_a + 1.0 / radius_b)
        force = (4.0 / 3.0) * self.effective_modulus * (effective_radius ** 0.5) * (penetration ** 1.5)
        return force
```

### 2.2 塑性接触モデル

接触圧力が降伏応力を超えると、材料は塑性変形します：

```python
class PlasticContactModel:
    """
    塑性接触モデル
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
        """塑性接触半径を計算"""
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
        """弾性接触半径を計算"""
        effective_radius = 1.0 / (1.0 / radius_a + 1.0 / radius_b)
        return (3 * normal_force * effective_radius / (4 * self.E)) ** (1.0 / 3.0)
```

### 2.3 粘弾性接触モデル

粘弾性材料は時間依存の応力-ひずみ関係を示します：

```python
class ViscoelasticContactModel:
    """
    粘弾性接触モデル
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
        """緩和率を計算"""
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
        粘弾性接触力を計算
        
        Kelvin-Voigt モデルを使用
        """
        effective_radius = 1.0 / (1.0 / radius_a + 1.0 / radius_b)
        
        elastic_force = (4.0 / 3.0) * self.E * (effective_radius ** 0.5) * (penetration ** 1.5)
        
        damping_force = (4.0 / 3.0) * self.eta * (effective_radius ** 0.5) * (loading_rate) * (penetration ** 0.5)
        
        relaxation_factor = 1 - np.exp(-time / self.tau)
        
        return elastic_force * relaxation_factor + damping_force
```

---

## 3. 摩擦モデル

### 3.1 クーロン摩擦

クーロン摩擦は乾いた摩擦を記述する基本モデルです：

$$F_f = \mu N$$

ここで $\mu$ は摩擦係数、$N$ は法線力です。

```python
class CoulombFriction:
    """
    クーロン摩擦モデル
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
        摩擦力ベクトルを計算
        
        パラメータ:
            normal_force: 法線力大小
            tangential_velocity: 接線速度ベクトル
            contact_point: 接触点位置
        
        戻り値:
            摩擦力ベクトル
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
        """静摩擦と動摩擦の間でスムーズに移行"""
        t = min(1.0, (speed - self.v_transition) / (self.v_transition * 10))
        return self.mu_s + t * (self.mu_d - self.mu_s)
```

### 3.2 転がり摩擦

転がり摩擦は回転する物体が表面と接触する際に発生します：

```python
class RollingFriction:
    """
    転がり摩擦モデル
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
        転がり摩擦トルクを計算
        
        公式: M_roll = C_rr * N * r * û
        ただし û は接触法線方向の単位ベクトル
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
        転がりによる減速度と角加速度を計算
        """
        linear_decel = -self.C_rr * normal_force * np.sign(velocity)
        
        rolling_torque = self.compute_rolling_torque(
            normal_force, wheel_radius, angular_velocity, np.array([0, 1, 0])
        )
        
        return linear_decel, rolling_torque
```

---

## 4. 衝突応答戦略

### 4.1 インパルス-Based 衝突応答

運動量保存に基づくインパルス法：

```python
class ImpulseBasedCollisionResponse:
    """
    インパルス-Based 衝突応答
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
        衝突インパルスを計算
        
        公式: j = -(1 + e) * v_rel · n / (1/m_a + 1/m_b)
        
        パラメータ:
            pos_a, pos_b: 衝突点位置
            vel_a, vel_b: 衝突前速度
            mass_a, mass_b: 質量
            normal: 接触法線ベクトル（A から B へ向く）
            restitution: 回復係数
        
        戻り値:
            インパルスベクトル（A に作用）、B には逆方向のインパルス
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
        インパルスを剛体に適用
        """
        body_a.velocity += impulse / body_a.mass
        
        r_a = contact_point - body_a.position
        body_a.angular_velocity += np.cross(r_a, impulse) / body_a.inertia
        
        body_b.velocity -= impulse / body_b.mass
        
        r_b = contact_point - body_b.position
        body_b.angular_velocity -= np.cross(r_b, impulse) / body_b.inertia
```

### 4.2 Penalty-Based 衝突応答

ばね-減衰モデルを使用した連続衝突応答：

```python
class PenaltyBasedCollisionResponse:
    """
    Penalty-Based 衝突応答（ばね-減衰モデル）
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
        penalty 接触力を計算
        
        公式: F = k * δ + c * δ̇
        ただし k は剛性、c は減衰係数
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
        """連続力から等価インパルスを計算"""
        return contact_force * dt
```

### 4.3 拘束-Based 衝突応答

拘束ソルバーを使用した多体衝突処理：

```python
class ConstraintBasedCollisionResponse:
    """
    拘束-Based 衝突応答
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
        接触拘束を作成
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
        すべての接触拘束を求解
        
        Gauss-Seidel 反復求解を使用
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
        """単一拘束を求解"""
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
    """接触拘束"""
    
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

## 5. 高速衝突と隧穿効果の処理

### 5.1 連続衝突検出 (Continuous Collision Detection, CCD)

物体の速度が速い場合、離散衝突検出では衝突を見逃す可能性があります（隧穿効果）。CCD は時間間隔内のすべての衝突が検出されることを保証します：

```python
class ContinuousCollisionDetection:
    """
    連続衝突検出
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
        衝突時間を計算
        
        パラメータ:
            pos_a_start: A の開始位置
            pos_a_end: A の終了位置
            vel_a: A の速度
            radius_a: A の半径
            pos_b: B の位置（静止と仮定）
            radius_b: B の半径
        
        戻り値:
            衝突時間（0 から 1 の間）、衝突がない場合は None
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
        スウィープテスト：運動経路上のすべての衝突を検出
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

### 5.2 サブステップ細分化

衝突検出精度を向上させるために、時間ステップを複数のサブステップに細分化：

```python
class SubstepCollisionSolver:
    """
    サブステップ衝突ソルバー
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
        サブステップで衝突を求解
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

## 6. 多体衝突システム

### 6.1 衝突グラフと衝突行列

```python
import numpy as np
from collections import defaultdict

class CollisionGraph:
    """
    衝突グラフは多体衝突関係を表現
    """
    
    def __init__(self):
        self.nodes = set()
        self.edges = defaultdict(set)
    
    def add_collision(self, body_a, body_b):
        """衝突エッジを追加"""
        self.nodes.add(body_a)
        self.nodes.add(body_b)
        self.edges[body_a].add(body_b)
        self.edges[body_b].add(body_a)
    
    def get_collision_groups(self) -> List[set]:
        """連結衝突グループを取得"""
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
    多体衝突システム
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
        すべての衝突を検出し解決
        """
        self.collision_graph = CollisionGraph()
        
        contacts = self._broad_phase(bodies)
        
        contacts = self._narrow_phase(contacts)
        
        contacts = self._filter_redundant_contacts(contacts)
        
        for contact in contacts:
            self._resolve_contact(contact)
        
        return contacts
    
    def _broad_phase(self, bodies: List['RigidBody']) -> List[tuple]:
        """広域フェーズ：BVH を使用して衝突不可能な物体をフィルタリング"""
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
        """狭域フェーズ：精密衝突検出"""
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
        """接触点情報を計算"""
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
        """冗長接触をフィルタリング"""
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
        """単一接触を解決"""
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
    """衝突接触"""
    
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

*このドキュメントは DYNAMICS_ENGINE の衝突システムサブモジュールです。*
*バージョン: v1.1*
