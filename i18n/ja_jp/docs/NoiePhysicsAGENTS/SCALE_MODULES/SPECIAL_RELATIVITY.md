# SPECIAL_RELATIVITY.md

## 特殊相対性理論 (PS-LR)

**スケール:** v > 0.1c  
**バージョン:** v1.0  
**状態:** 検証済み

---

## 概要

本文書は**特殊相対性理論**の物理フレームワークを処理する。NoiePhysicsAGENTS.md §1の物理スケール権限レベル定義に従い、PS-LRは相対論的効果が顕著なスケールを表し、光速に近い速度で運動する時に特殊相対性理論を使用する必要がある。

---

## 重要な安全と真理プロトコル

> **CRITICAL SAFETY & TRUTH PROTOCOL:**
> 1. AXIOMS.mdのPT-AX17（光速不変）、PT-AX18（時間遅延）を遵守
> 2. 特殊相対性理論は十分に検証された理論である
> 3. v > 0.1cでの相対論的効果に注意
> 4. 監査：全ての異常をPHYSICS_AUDIT_TRAILに記録

---

## 1. 時空構造

### 1.1 ミンコフスキー時空

$$ds^2 = -c^2dt^2 + dx^2 + dy^2 + dz^2$$

```python
class MinkowskiMetric:
    """
    ミンコフスキー計量
    """
    
    signature = [-1, 1, 1, 1]
    
    def interval(self, event1: Event, event2: Event) -> float:
        """時空間隔を計算"""
        dt = event2.t - event1.t
        dx = event2.x - event1.x
        dy = event2.y - event1.y
        dz = event2.z - event1.z
        return -c**2 * dt**2 + dx**2 + dy**2 + dz**2
```

### 1.2 ローレンツ変換

$$x'^\mu = \Lambda^\mu_{\ \nu} x^\nu$$

```python
class LorentzTransformation:
    """
    ローレンツ変換
    """
    
    def lorentz_factor(self, velocity: float) -> float:
        """ローレンツ因子を計算"""
        beta = velocity / c
        return 1 / np.sqrt(1 - beta**2)
    
    def transform_coordinates(
        self,
        event: Event,
        velocity: Vector3D
    ) -> Event:
        """座標変換"""
        gamma = self.lorentz_factor(velocity.magnitude())
        # ローレンツ変換行列
        pass
```

---

## 2. 相対論的効果

### 2.1 時間遅延

$$\Delta t' = \gamma \Delta t$$

```python
class TimeDilation:
    """
    時間遅延
    """
    
    def dilated_time(
        self,
        proper_time: float,
        velocity: float
    ) -> float:
        """遅延後の時間を計算"""
        gamma = 1 / np.sqrt(1 - (velocity / c)**2)
        return proper_time * gamma
```

### 2.2 長さの収縮

$$L' = \frac{L}{\gamma}$$

### 2.3 質量エネルギー等価性

$$E = mc^2 = \gamma m_0 c^2$$

```python
class MassEnergyEquivalence:
    """
    質量エネルギー等価性
    """
    
    def relativistic_energy(
        self,
        rest_mass: float,
        velocity: float
    ) -> float:
        """相対論的エネルギーを計算"""
        gamma = 1 / np.sqrt(1 - (velocity / c)**2)
        return gamma * rest_mass * c**2
    
    def kinetic_energy(
        self,
        rest_mass: float,
        velocity: float
    ) -> float:
        """運動エネルギーを計算"""
        gamma = 1 / np.sqrt(1 - (velocity / c)**2)
        return (gamma - 1) * rest_mass * c**2
```

---

## 3. 四次元表現

### 3.1 四元ベクトル

$$A^\mu = (A^0, \mathbf{A})$$

### 3.2 四元運動量

$$p^\mu = (E/c, \mathbf{p})$$

---

## 4. 他のスケールとのインターフェース

### 4.1 ニュートン力学とのインターフェース

```
特殊相対性理論 → ニュートン力学：
- 低速極限 v << c：γ ≈ 1
- 古典力学に回帰
```

---

*本文書は特殊相対性理論の物理フレームワークを処理する。*
*特殊相対性理論は現代物理学の土台であり、実験と高度に一致する。*
