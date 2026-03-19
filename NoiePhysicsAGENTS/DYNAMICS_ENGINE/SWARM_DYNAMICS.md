# SWARM_DYNAMICS.md

## L3 - 群體動力學

> **WARNING:** 本模組是 DYNAMICS_ENGINE 的群體動力學子模組。
> **版本:** v1.1

---

## 概述

本文檔定義 NoiePhysicsAGENTS 的群體動力學系統，處理多智能體系統的協調運動。系統涵蓋從簡單的 Boids 模型到複雜的分布式共識演算法，支援自組織行為和湧現現象的模擬。

---

## 1. 群體表示與狀態管理

### 1.1 群體狀態

```python
import numpy as np
from typing import List, Dict, Set, Optional, Tuple
from dataclasses import dataclass, field
from enum import Enum

class SwarmPhase(Enum):
    """群體相位狀態"""
    DISORDERED = "disordered"           # 無序：隨機運動
    COHERENT = "coherent"                # 相干：同步運動
    POLARIZED = "polarized"              # 極化：朝向一致
    CLUSTERED = "clustered"              # 聚集：形成團簇
    FLOCKING = "flocking"                # 群飛：經典 Boids 行為


@dataclass
class AgentState:
    """單個智能體狀態"""
    id: str
    position: np.ndarray
    velocity: np.ndarray
    acceleration: np.ndarray
    heading: float                      # 朝向角
    angular_velocity: float              # 角速度
    
    neighbors: Set[str] = field(default_factory=set)
    local_environment: Dict = field(default_factory=dict)
    
    energy: float = 100.0
    health: float = 1.0
    
    task_assignment: Optional[str] = None
    belief_state: Dict = field(default_factory=dict)


@dataclass
class SwarmState:
    """完整群體狀態"""
    agents: Dict[str, AgentState] = field(default_factory=dict)
    connectivity_graph: np.ndarray = None  # 鄰接矩陣
    
    global_phase: SwarmPhase = SwarmPhase.DISORDERED
    
    center_of_mass: np.ndarray = None
    average_velocity: np.ndarray = None
    
    order_parameter: float = 0.0         # 秩序參數
    polarization: float = 0.0            # 極化程度
    
    consensus_value: Optional[float] = None
    consensus_confidence: float = 0.0
    
    timestamp: float = 0.0
    
    def compute_derived_quantities(self):
        """計算導出量"""
        if not self.agents:
            return
        
        positions = np.array([a.position for a in self.agents.values()])
        velocities = np.array([a.velocity for a in self.agents.values()])
        
        self.center_of_mass = np.mean(positions, axis=0)
        self.average_velocity = np.mean(velocities, axis=0)
        
        self.order_parameter = self._compute_order_parameter(velocities)
        self.polarization = self._compute_polarization(velocities)
    
    def _compute_order_parameter(self, velocities: np.ndarray) -> float:
        """計算秩序參數 (0-1)"""
        if len( velocities) == 0:
            return 0.0
        
        magnitudes = np.linalg.norm(velocities, axis=1)
        if np.sum(magnitudes) < 1e-6:
            return 0.0
        
        normalized_vels = velocities / magnitudes[:, np.newaxis]
        avg_direction = np.mean(normalized_vels, axis=0)
        
        return np.linalg.norm(avg_direction)
    
    def _compute_polarization(self, velocities: np.ndarray) -> float:
        """計算極化程度"""
        return self._compute_order_parameter(velocities)
```

### 1.2 連接拓撲管理

```python
class ConnectivityTopology:
    """
    連接拓撲管理
    """
    
    def __init__(
        self,
        communication_range: float = 10.0,
        topology_type: str = "distance_based"
    ):
        self.communication_range = communication_range
        self.topology_type = topology_type
        self.adjacency_matrix: np.ndarray = None
        self.distance_matrix: np.ndarray = None
    
    def update_connectivity(
        self,
        agents: Dict[str, AgentState]
    ) -> np.ndarray:
        """更新連接矩陣"""
        n = len(agents)
        agent_list = list(agents.values())
        
        self.distance_matrix = np.zeros((n, n))
        
        for i in range(n):
            for j in range(i + 1, n):
                dist = np.linalg.norm(
                    agent_list[i].position - agent_list[j].position
                )
                self.distance_matrix[i, j] = dist
                self.distance_matrix[j, i] = dist
        
        if self.topology_type == "distance_based":
            self.adjacency_matrix = (
                self.distance_matrix < self.communication_range
            ).astype(float)
            np.fill_diagonal(self.adjacency_matrix, 0)
        
        elif self.topology_type == "k_nearest":
            self.adjacency_matrix = self._k_nearest_matrix(n)
        
        elif self.topology_type == "mechanism_range":
            self.adjacency_matrix = self._mechanism_range_matrix()
        
        return self.adjacency_matrix
    
    def _k_nearest_matrix(self, n: int, k: int = 4) -> np.ndarray:
        """K近鄰連接矩陣"""
        adj = np.zeros((n, n))
        
        for i in range(n):
            distances = self.distance_matrix[i]
            nearest_indices = np.argsort(distances)[1:k+1]
            adj[i, nearest_indices] = 1
        
        return adj
    
    def _mechanism_range_matrix(self) -> np.ndarray:
        """感知範圍內連接"""
        return (self.distance_matrix < self.communication_range).astype(float)
    
    def get_neighbors(self, agent_id: str, agents: Dict[str, AgentState]) -> List[AgentState]:
        """獲取智能體的所有鄰居"""
        agent_list = list(agents.keys())
        if agent_id not in agent_list:
            return []
        
        idx = agent_list.index(agent_id)
        neighbor_indices = np.where(self.adjacency_matrix[idx] > 0)[0]
        
        return [agents[agent_list[i]] for i in neighbor_indices]
```

---

## 2. 群體行為模型

### 2.1 Boids 模型

經典的 Boids 模型包含三個核心行為：凝聚、分離、對齊。

```python
class BoidsModel:
    """
    Boids 群體行為模型
    
    核心規則：
    - 凝聚 (Cohesion): 朝向鄰居中心移動
    - 分離 (Separation): 避免與鄰居過度接近
    - 對齊 (Alignment): 匹配鄰居的速度方向
    """
    
    def __init__(
        self,
        perception_radius: float = 5.0,
        separation_radius: float = 2.0,
        
        cohesion_weight: float = 1.0,
        separation_weight: float = 1.5,
        alignment_weight: self = 1.0,
        
        max_speed: float = 10.0,
        max_force: float = 5.0
    ):
        self.perception_radius = perception_radius
        self.separation_radius = separation_radius
        
        self.cohesion_weight = cohesion_weight
        self.separation_weight = separation_weight
        self.alignment_weight = alignment_weight
        
        self.max_speed = max_speed
        self.max_force = max_force
    
    def compute_flocking_force(
        self,
        agent: AgentState,
        neighbors: List[AgentState]
    ) -> np.ndarray:
        """計算 Boids  flocking 力"""
        if not neighbors:
            return np.zeros(3)
        
        cohesion_force = self._compute_cohesion(agent, neighbors)
        separation_force = self._compute_separation(agent, neighbors)
        alignment_force = self._compute_alignment(agent, neighbors)
        
        total_force = (
            self.cohesion_weight * cohesion_force +
            self.separation_weight * separation_force +
            self.alignment_weight * alignment_force
        )
        
        return self._limit_force(total_force)
    
    def _compute_cohesion(
        self,
        agent: AgentState,
        neighbors: List[AgentState]
    ) -> np.ndarray:
        """計算凝聚力：朝向鄰居中心"""
        if not neighbors:
            return np.zeros(3)
        
        center = np.mean([n.position for n in neighbors], axis=0)
        
        desired = center - agent.position
        
        if np.linalg.norm(desired) > 1e-6:
            desired = desired / np.linalg.norm(desired) * self.max_speed
        
        steer = desired - agent.velocity
        
        return self._limit_force(steer)
    
    def _compute_separation(
        self,
        agent: AgentState,
        neighbors: List[AgentState]
    ) -> np.ndarray:
        """計算分離力：遠離過近的鄰居"""
        steer = np.zeros(3)
        count = 0
        
        for neighbor in neighbors:
            distance = np.linalg.norm(agent.position - neighbor.position)
            
            if 0 < distance < self.separation_radius:
                diff = agent.position - neighbor.position
                diff = diff / (distance + 1e-6)
                steer += diff
                count += 1
        
        if count > 0:
            steer = steer / count
            
            if np.linalg.norm(steer) > 1e-6:
                steer = steer / np.linalg.norm(steer) * self.max_speed
                steer = steer - agent.velocity
        
        return self._limit_force(steer)
    
    def _compute_alignment(
        self,
        agent: AgentState,
        neighbors: List[AgentState]
    ) -> np.ndarray:
        """計算對齊力：匹配鄰居速度"""
        avg_velocity = np.mean([n.velocity for n in neighbors], axis=0)
        
        if np.linalg.norm(avg_velocity) > 1e-6:
            desired = avg_velocity / np.linalg.norm(avg_velocity) * self.max_speed
        else:
            desired = np.zeros(3)
        
        steer = desired - agent.velocity
        
        return self._limit_force(steer)
    
    def _limit_force(self, force: np.ndarray) -> np.ndarray:
        """限制最大力"""
        magnitude = np.linalg.norm(force)
        
        if magnitude > self.max_force:
            force = force / magnitude * self.max_force
        
        return force
    
    def limit_speed(self, velocity: np.ndarray) -> np.ndarray:
        """限制最大速度"""
        speed = np.linalg.norm(velocity)
        
        if speed > self.max_speed:
            velocity = velocity / speed * self.max_speed
        
        return velocity
```

### 2.2 粒子群優化 (PSO)

```python
class ParticleSwarmOptimizer:
    """
    粒子群優化演算法
    
    每個粒子維護：
    - 個人最佳位置 (pbest)
    - 群體最佳位置 (gbest)
    
    速度更新公式：
    v = w * v + c1 * r1 * (pbest - x) + c2 * r2 * (gbest - x)
    """
    
    def __init__(
        self,
        num_particles: int = 30,
        inertia_weight: float = 0.7,
        cognitive_coefficient: float = 1.5,
        social_coefficient: float = 1.5,
        max_velocity: float = 2.0
    ):
        self.n = num_particles
        self.w = inertia_weight
        self.c1 = cognitive_coefficient
        self.c2 = social_coefficient
        self.max_v = max_velocity
        
        self.particles: List[AgentState] = []
        self.global_best_position: Optional[np.ndarray] = None
        self.global_best_fitness: float = float('-inf')
    
    def initialize(
        self,
        search_space_min: np.ndarray,
        search_space_max: np.ndarray,
        objective_function
    ):
        """初始化粒子群"""
        dim = len(search_space_min)
        
        for i in range(self.n):
            position = np.random.uniform(search_space_min, search_space_max)
            velocity = np.random.uniform(-self.max_v, self.max_v, dim)
            
            agent = AgentState(
                id=f"pso_{i}",
                position=position,
                velocity=velocity,
                acceleration=np.zeros(dim),
                heading=0.0,
                angular_velocity=0.0
            )
            
            agent.belief_state['personal_best_position'] = position.copy()
            agent.belief_state['personal_best_fitness'] = float('-inf')
            
            self.particles.append(agent)
    
    def optimize(
        self,
        objective_function,
        max_iterations: int = 100,
        tolerance: float = 1e-6
    ) -> Tuple[np.ndarray, float]:
        """執行優化"""
        for iteration in range(max_iterations):
            for particle in self.particles:
                fitness = objective_function(particle.position)
                
                pbest = particle.belief_state['personal_best_fitness']
                if fitness > pbest:
                    particle.belief_state['personal_best_position'] = particle.position.copy()
                    particle.belief_state['personal_best_fitness'] = fitness
                
                if fitness > self.global_best_fitness:
                    self.global_best_position = particle.position.copy()
                    self.global_best_fitness = fitness
            
            if self.global_best_fitness > (1 - tolerance):
                break
            
            for particle in self.particles:
                self._update_velocity(particle)
                self._update_position(particle)
        
        return self.global_best_position, self.global_best_fitness
    
    def _update_velocity(self, particle: AgentState):
        """更新粒子速度"""
        r1, r2 = np.random.rand(2)
        
        pbest = particle.belief_state['personal_best_position']
        
        cognitive = self.c1 * r1 * (pbest - particle.position)
        social = self.c2 * r2 * (self.global_best_position - particle.position)
        
        particle.velocity = (
            self.w * particle.velocity +
            cognitive +
            social
        )
        
        speed = np.linalg.norm(particle.velocity)
        if speed > self.max_v:
            particle.velocity = particle.velocity / speed * self.max_v
    
    def _update_position(self, particle: AgentState):
        """更新粒子位置"""
        particle.position = particle.position + particle.velocity
```

---

## 3. 共識形成演算法

### 3.1 分布式共識

```python
class DistributedConsensus:
    """
    分布式共識演算法
    
    使用平均共識協議：
    x_i(t+1) = x_i(t) + ε * Σ(x_j(t) - x_i(t))
    
    收斂條件：
    - 連通圖
    - 雙隨機權重矩陣
    """
    
    def __init__(
        self,
        convergence_rate: float = 0.1,
        consensus_threshold: float = 1e-4
    ):
        self.epsilon = convergence_rate
        self.threshold = consensus_threshold
    
    def compute_consensus_update(
        self,
        agent: AgentState,
        neighbors: List[AgentState],
        adjacency_weights: np.ndarray
    ) -> np.ndarray:
        """計算共識更新"""
        current_value = agent.belief_state.get('consensus_value', 0.0)
        
        neighbor_sum = np.zeros_like(current_value) if isinstance(current_value, np.ndarray) else 0.0
        weight_sum = 0.0
        
        for i, neighbor in enumerate(neighbors):
            neighbor_value = neighbor.belief_state.get('consensus_value', 0.0)
            weight = adjacency_weights[i]
            
            neighbor_sum += weight * (neighbor_value - current_value)
            weight_sum += weight
        
        if weight_sum > 0:
            delta = self.epsilon * neighbor_sum
        else:
            delta = 0
        
        return delta
    
    def check_consensus(
        self,
        agents: Dict[str, AgentState]
    ) -> Tuple[bool, float, Optional[float]]:
        """檢查是否達成共識"""
        values = [
            agent.belief_state.get('consensus_value', 0.0)
            for agent in agents.values()
        ]
        
        if isinstance(values[0], np.ndarray):
            mean_value = np.mean(values, axis=0)
            variance = np.mean([np.linalg.norm(v - mean_value)**2 for v in values])
            converged = variance < self.threshold
            confidence = 1.0 / (1.0 + variance)
            return converged, confidence, mean_value
        else:
            mean_value = np.mean(values)
            variance = np.var(values)
            converged = variance < self.threshold
            confidence = 1.0 / (1.0 + variance)
            return converged, confidence, mean_value
    
    def initialize_consensus(
        self,
        agents: Dict[str, AgentState],
        initial_values: Dict[str, float]
    ):
        """初始化共識值"""
        for agent_id, value in initial_values.items():
            if agent_id in agents:
                agents[agent_id].belief_state['consensus_value'] = value
```

### 3.2 投票共識

```python
class VotingConsensus:
    """
    投票共識機制
    """
    
    def __init__(
        self,
        voting_threshold: float = 0.5,
        rounds: int = 10
    ):
        self.threshold = voting_threshold
        self.max_rounds = rounds
    
    def vote(
        self,
        agents: Dict[str, AgentState],
        proposition: str
    ) -> Dict[str, bool]:
        """執行投票"""
        votes = {}
        
        for agent_id, agent in agents.items():
            vote = self._individual_vote(agent, proposition)
            votes[agent_id] = vote
        
        return votes
    
    def _individual_vote(
        self,
        agent: AgentState,
        proposition: str
    ) -> bool:
        """個體投票邏輯"""
        belief = agent.belief_state.get(f'belief_{proposition}', 0.5)
        
        noise = np.random.normal(0, 0.1)
        decision_value = belief + noise
        
        return decision_value > self.threshold
    
    def count_votes(
        self,
        votes: Dict[str, bool]
    ) -> Tuple[bool, float]:
        """計數投票結果"""
        total = len(votes)
        if total == 0:
            return False, 0.0
        
        yes_votes = sum(votes.values())
        yes_ratio = yes_votes / total
        
        consensus_reached = yes_ratio > self.threshold
        
        return consensus_reached, yes_ratio
```

---

## 4. 群體通訊協定

### 4.1 訊息傳遞

```python
from dataclasses import dataclass
from typing import Any, Optional
import time

@dataclass
class SwarmMessage:
    """群體訊息"""
    sender_id: str
    receiver_id: Optional[str]  # None 表示廣播
    message_type: str
    payload: Any
    timestamp: float
    ttl: int = 3  # 生存時間
    
    def is_expired(self) -> bool:
        return self.ttl <= 0


class SwarmCommunicationProtocol:
    """
    群體通訊協定
    """
    
    def __init__(
        self,
        bandwidth_limit: float = 1000.0,
        message_delay: float = 0.01
    ):
        self.bandwidth_limit = bandwidth_limit
        self.message_delay = message_delay
        
        self.message_queue: List[SwarmMessage] = []
        self.delivered_messages: Set[str] = set()
    
    def send_message(
        self,
        sender: AgentState,
        receiver_id: Optional[str],
        message_type: str,
        payload: Any,
        ttl: int = 3
    ) -> SwarmMessage:
        """發送訊息"""
        message_id = f"{sender.id}_{time.time()}_{message_type}"
        
        message = SwarmMessage(
            sender_id=sender.id,
            receiver_id=receiver_id,
            message_type=message_type,
            payload=payload,
            timestamp=time.time(),
            ttl=ttl
        )
        
        self.message_queue.append(message)
        
        return message
    
    def broadcast(
        self,
        sender: AgentState,
        message_type: str,
        payload: Any
    ) -> SwarmMessage:
        """廣播訊息"""
        return self.send_message(sender, None, message_type, payload)
    
    def deliver_messages(
        self,
        agents: Dict[str, AgentState]
    ) -> Dict[str, List[SwarmMessage]]:
        """遞送訊息"""
        delivered = {agent_id: [] for agent_id in agents.keys()}
        
        for message in self.message_queue[:]:
            if message.is_expired():
                self.message_queue.remove(message)
                continue
            
            target_agents = self._get_target_agents(message, agents)
            
            for agent_id in target_agents:
                delivered[agent_id].append(message)
                self.delivered_messages.add(f"{message.sender_id}_{message.timestamp}")
            
            message.ttl -= 1
        
        return delivered
    
    def _get_target_agents(
        self,
        message: SwarmMessage,
        agents: Dict[str, AgentState]
    ) -> List[str]:
        """獲取目標智能體"""
        if message.receiver_id is None:
            return [aid for aid in agents.keys() if aid != message.sender_id]
        elif message.receiver_id in agents:
            return [message.receiver_id]
        else:
            return []
```

### 4.2 訊息類型

```python
class MessageTypes:
    """預定義訊息類型"""
    
    POSITION_UPDATE = "position_update"
    VELOCITY_SYNC = "velocity_sync"
    THREAT_ALERT = "threat_alert"
    TASK_ASSIGNMENT = "task_assignment"
    CONSENSUS_PROPOSAL = "consensus_proposal"
    CONSENSUS_VOTE = "consensus_vote"
    ENERGY_REQUEST = "energy_request"
    FORMATION_COMMAND = "formation_command"


@dataclass
class PositionUpdateMessage:
    """位置更新訊息"""
    sender_id: str
    position: np.ndarray
    velocity: np.ndarray
    timestamp: float


@dataclass
class ThreatAlertMessage:
    """威脅警報訊息"""
    sender_id: str
    threat_position: np.ndarray
    threat_type: str
    threat_level: float
    timestamp: float
```

---

## 5. 群體感知與感知融合

### 5.1 局部感知

```python
class LocalPerception:
    """
    局部感知系統
    """
    
    def __init__(
        self,
        vision_range: float = 20.0,
        fov_angle: float = 270.0  # 度
    ):
        self.vision_range = vision_range
        self.fov_angle = np.radians(fov_angle)
    
    def perceive_environment(
        self,
        agent: AgentState,
        all_agents: Dict[str, AgentState],
        obstacles: List
    ) -> Dict:
        """感知環境"""
        perception = {
            'visible_agents': [],
            'nearby_obstacles': [],
            'perceived_threats': [],
            'resource_locations': []
        }
        
        for other_id, other in all_agents.items():
            if other_id == agent.id:
                continue
            
            if self._is_visible(agent, other):
                perception['visible_agents'].append(other)
                
                if self._is_threat(agent, other):
                    perception['perceived_threats'].append(other)
        
        for obstacle in obstacles:
            if self._is_in_range(agent.position, obstacle.position):
                perception['nearby_obstacles'].append(obstacle)
        
        return perception
    
    def _is_visible(
        self,
        observer: AgentState,
        target: AgentState
    ) -> bool:
        """檢查目標是否可見"""
        direction_to_target = target.position - observer.position
        distance = np.linalg.norm(direction_to_target)
        
        if distance > self.vision_range:
            return False
        
        if distance < 1e-6:
            return True
        
        direction_to_target = direction_to_target / distance
        
        heading_vector = np.array([
            np.cos(observer.heading),
            np.sin(observer.heading),
            0
        ])
        
        dot_product = np.dot(heading_vector, direction_to_target)
        angle = np.arccos(np.clip(dot_product, -1, 1))
        
        return angle < self.fov_angle / 2
    
    def _is_in_range(
        self,
        position: np.ndarray,
        target_position: np.ndarray
    ) -> bool:
        """檢查是否在感知範圍內"""
        return np.linalg.norm(position - target_position) < self.vision_range
    
    def _is_threat(
        self,
        agent: AgentState,
        other: AgentState
    ) -> bool:
        """判斷是否為威脅"""
        relative_velocity = agent.velocity - other.velocity
        relative_position = other.position - agent.position
        
        approach_speed = -np.dot(relative_velocity, relative_position)
        
        return (approach_speed > 0 and 
                np.linalg.norm(relative_position) < self.vision_range * 0.5)
```

### 5.2 感知融合

```python
class SensorFusion:
    """
    感知融合模組
    """
    
    def __init__(self, fusion_method: str = "kalman"):
        self.fusion_method = fusion_method
    
    def fuse_position_estimates(
        self,
        local_estimate: np.ndarray,
        neighbor_estimates: List[Tuple[np.ndarray, float]]
    ) -> np.ndarray:
        """
        融合位置估計
        
        參數:
            local_estimate: 本地估計
            neighbor_estimates: 鄰居估計列表 [(估計值, 置信度), ...]
        
        返回:
            融合後的估計
        """
        if not neighbor_estimates:
            return local_estimate
        
        if self.fusion_method == "kalman":
            return self._kalman_fusion(local_estimate, neighbor_estimates)
        elif self.fusion_method == "weighted_average":
            return self._weighted_average_fusion(local_estimate, neighbor_estimates)
        elif self.fusion_method == "covariance_intersection":
            return self._covariance_intersection_fusion(local_estimate, neighbor_estimates)
        else:
            return local_estimate
    
    def _kalman_fusion(
        self,
        local_estimate: np.ndarray,
        neighbor_estimates: List[Tuple[np.ndarray, float]]
    ) -> np.ndarray:
        """卡爾曼濾波融合"""
        local_variance = 1.0
        
        estimates = [local_estimate]
        variances = [local_variance]
        
        for estimate, confidence in neighbor_estimates:
            variance = 1.0 / (confidence + 1e-6)
            estimates.append(estimate)
            variances.append(variance)
        
        weights = [1.0 / v for v in variances]
        weight_sum = sum(weights)
        
        normalized_weights = [w / weight_sum for w in weights]
        
        fused = np.zeros_like(local_estimate)
        for i, estimate in enumerate(estimates):
            fused += normalized_weights[i] * estimate
        
        return fused
    
    def _weighted_average_fusion(
        self,
        local_estimate: np.ndarray,
        neighbor_estimates: List[Tuple[np.ndarray, float]]
    ) -> np.ndarray:
        """加權平均融合"""
        total_weight = 1.0
        
        fused = local_estimate.copy()
        
        for estimate, confidence in neighbor_estimates:
            fused += confidence * estimate
            total_weight += confidence
        
        return fused / total_weight
    
    def _covariance_intersection_fusion(
        self,
        local_estimate: np.ndarray,
        neighbor_estimates: List[Tuple[np.ndarray, float]]
    ) -> np.ndarray:
        """協方差交叉融合"""
        local_covariance = np.eye(3) * 1.0
        
        fused_estimate = local_estimate.copy()
        fused_covariance = local_covariance.copy()
        
        for estimate, confidence in neighbor_estimates:
            omega = 1.0 / (confidence + 1e-6)
            
            innovation = estimate - fused_estimate
            
            combined_covariance = np.linalg.inv(
                np.linalg.inv(fused_covariance) + omega * np.linalg.inv(local_covariance)
            )
            
            fused_estimate = combined_covariance @ (
                np.linalg.inv(fused_covariance) @ fused_estimate +
                omega * np.linalg.inv(local_covariance) @ estimate
            )
            
            fused_covariance = combined_covariance
        
        return fused_estimate
```

---

## 6. 自組織與湧現行為

### 6.1 相變模型

```python
class SwarmPhaseTransition:
    """
    群體相變模型
    
    模擬群體在不同相態之間的轉換：
    - 氣態 (Gas): 無序，獨立運動
    - 液態 (Liquid): 弱耦合，局部有序
    - 固態 (Solid): 強耦合，全域有序
    """
    
    PHASES = {
        'gas': {'coupling': 0.0, 'order': 0.0},
        'liquid': {'coupling': 0.5, 'order': 0.5},
        'solid': {'coupling': 1.0, 'order': 1.0}
    }
    
    def __init__(
        self,
        phase_transition_temperature: float = 0.5,
        coupling_strength: float = 1.0
    ):
        self.T_c = phase_transition_temperature
        self.J = coupling_strength
    
    def compute_coupling_strength(
        self,
        temperature: float,
        phase: str
    ) -> float:
        """
        計算耦合強度
        
        使用平均場近似：
        J_eff = J * (1 - T/T_c)
        """
        if temperature < self.T_c:
            return self.J * (1 - temperature / self.T_c)
        else:
            return 0.0
    
    def detect_phase_transition(
        self,
        swarm_state: SwarmState
    ) -> str:
        """檢測當前相態"""
        order_param = swarm_state.order_parameter
        
        if order_param > 0.8:
            return 'solid'
        elif order_param > 0.4:
            return 'liquid'
        else:
            return 'gas'
    
    def compute_phase_boundary(
        self,
        density: float,
        temperature: float
    ) -> Dict[str, float]:
        """計算相邊界"""
        critical_density = self.T_c / self.J
        
        return {
            'gas_liquid': density < critical_density * 0.5,
            'liquid_solid': density > critical_density and temperature < self.T_c * 0.8,
            'gas_solid': density > critical_density * 1.5 and temperature < self.T_c * 0.3
        }
```

### 6.2 湧現行為檢測

```python
class EmergenceDetector:
    """
    湧現行為檢測器
    """
    
    def __init__(self):
        self.behavior_history: List[Dict] = []
    
    def detect_emergence(
        self,
        swarm_state: SwarmState,
        individual_behaviors: Dict[str, np.ndarray]
    ) -> Dict[str, float]:
        """
        檢測湧現行為
        
        湧現指標：
        - 資訊熵：系統無序程度
        - 相關性：智能體之間的關聯程度
        - 標度律：不同層級行為的一致性
        """
        emergence_indicators = {}
        
        emergence_indicators['information_entropy'] = self._compute_entropy(
            swarm_state
        )
        
        emergence_indicators['spatial_correlation'] = self._compute_spatial_correlation(
            swarm_state
        )
        
        emergence_indicators['velocity_correlation'] = self._compute_velocity_correlation(
            swarm_state
        )
        
        emergence_indicators['collective_pattern'] = self._detect_collective_pattern(
            swarm_state
        )
        
        return emergence_indicators
    
    def _compute_entropy(self, swarm_state: SwarmState) -> float:
        """計算空間熵"""
        if not swarm_state.agents:
            return 0.0
        
        positions = np.array([a.position for a in swarm_state.agents.values()])
        
        min_bounds = positions.min(axis=0) - 1.0
        max_bounds = positions.max(axis=0) + 1.0
        
        bins = 10
        hist, _ = np.histogramdd(positions, bins=bins)
        
        hist = hist / (hist.sum() + 1e-6)
        
        entropy = -np.sum(hist * np.log(hist + 1e-6))
        
        return entropy
    
    def _compute_spatial_correlation(self, swarm_state: SwarmState) -> float:
        """計算空間相關性"""
        positions = list(swarm_state.agents.values())
        n = len(positions)
        
        if n < 2:
            return 0.0
        
        distances = []
        for i in range(n):
            for j in range(i + 1, n):
                d = np.linalg.norm(positions[i].position - positions[j].position)
                distances.append(d)
        
        mean_distance = np.mean(distances)
        std_distance = np.std(distances)
        
        return 1.0 / (1.0 + std_distance / (mean_distance + 1e-6))
    
    def _compute_velocity_correlation(self, swarm_state: SwarmState) -> float:
        """計算速度相關性"""
        velocities = np.array([a.velocity for a in swarm_state.agents.values()])
        
        if len(velocities) < 2:
            return 0.0
        
        normalized_vels = velocities / (np.linalg.norm(velocities, axis=1, keepdims=True) + 1e-6)
        
        correlation_matrix = normalized_vels @ normalized_vels.T
        
        n = len(velocities)
        upper_triangle = correlation_matrix[np.triu_indices(n, k=1)]
        
        return np.mean(upper_triangle)
    
    def _detect_collective_pattern(self, swarm_state: SwarmState) -> str:
        """檢測集體模式"""
        order = swarm_state.order_parameter
        polarization = swarm_state.polarization
        
        if order > 0.8 and polarization > 0.7:
            return "highly_coordinated"
        elif order > 0.5:
            return "moderately_organized"
        elif polarization > 0.5:
            return "directional_movement"
        else:
            return "disorganized"
```

---

## 7. 群體任務分配

### 7.1 市場機制任務分配

```python
class MarketBasedTaskAllocation:
    """
    基於市場機制的任務分配
    """
    
    def __init__(self):
        self.auction_bids: Dict[str, List['Bid']] = {}
    
    def auction_task(
        self,
        task: 'Task',
        available_agents: List[AgentState]
    ) -> str:
        """拍賣任務"""
        bids = []
        
        for agent in available_agents:
            bid = self._compute_bid(agent, task)
            bids.append(bid)
        
        winning_bid = min(bids, key=lambda b: b.cost)
        
        return winning_bid.agent_id
    
    def _compute_bid(self, agent: AgentState, task: 'Task') -> 'Bid':
        """計算投標"""
        distance_to_task = np.linalg.norm(agent.position - task.position)
        
        travel_time = distance_to_task / (np.linalg.norm(agent.velocity) + 1e-6)
        
        energy_cost = distance_to_task * agent.energy_cost_per_unit
        
        capability_match = self._evaluate_capability(agent, task)
        
        total_cost = (
            travel_time * 1.0 +
            energy_cost * 0.5 +
            (1 - capability_match) * 10.0
        )
        
        return Bid(
            agent_id=agent.id,
            task_id=task.id,
            cost=total_cost,
            capability_match=capability_match
        )
    
    def _evaluate_capability(
        self,
        agent: AgentState,
        task: 'Task
    ) -> float:
        """評估能力匹配度"""
        required_capabilities = task.required_capabilities
        
        if not required_capabilities:
            return 1.0
        
        agent_capabilities = agent.belief_state.get('capabilities', {})
        
        match_scores = []
        for cap, required_level in required_capabilities.items():
            agent_level = agent_capabilities.get(cap, 0.0)
            match_scores.append(min(1.0, agent_level / (required_level + 1e-6)))
        
        return np.mean(match_scores) if match_scores else 0.0


@dataclass
class Bid:
    """投標"""
    agent_id: str
    task_id: str
    cost: float
    capability_match: float


@dataclass
class Task:
    """任務"""
    id: str
    position: np.ndarray
    required_capabilities: Dict[str, float]
    deadline: float
    reward: float
```

### 7.2 行為式任務分配

```python
class BehavioralTaskAllocation:
    """
    基於行為的任務分配
    """
    
    def __init__(self):
        self.task_queue: List[Task] = []
        self.assignment_history: List[Dict] = []
    
    def assign_tasks(
        self,
        agents: Dict[str, AgentState],
        tasks: List[Task]
    ) -> Dict[str, Optional[Task]]:
        """分配任務"""
        assignments = {agent_id: None for agent_id in agents.keys()}
        
        sorted_tasks = sorted(tasks, key=lambda t: t.deadline)
        
        for task in sorted_tasks:
            best_agent = self._select_best_agent(agents, task, assignments)
            
            if best_agent is not None:
                assignments[best_agent] = task
                agents[best_agent].task_assignment = task.id
        
        self.assignment_history.append({
            'tasks': len(tasks),
            'assignments': sum(1 for a in assignments.values() if a is not None)
        })
        
        return assignments
    
    def _select_best_agent(
        self,
        agents: Dict[str, AgentState],
        task: Task,
        current_assignments: Dict[str, Optional[Task]]
    ) -> Optional[str]:
        """選擇最佳智能體"""
        candidates = []
        
        for agent_id, agent in agents.items():
            if current_assignments[agent_id] is not None:
                continue
            
            if agent.energy < 0.2:
                continue
            
            distance = np.linalg.norm(agent.position - task.position)
            
            urgency = 1.0 / (task.deadline + 1e-6)
            
            score = -(distance * 0.5 + urgency * 10.0)
            
            candidates.append((agent_id, score))
        
        if not candidates:
            return None
        
        candidates.sort(key=lambda x: x[1])
        
        return candidates[0][0]
```

---

*本文檔是 DYNAMICS_ENGINE 的群體動力學子模組。*
*版本: v1.1*
