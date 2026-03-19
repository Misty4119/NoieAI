# SPECIAL_RELATIVITY.md

## 狹義相對論 (PS-LR)

**尺度：** v > 0.1c  
**版本：** v1.0  
**狀態：** 驗證性

---

## 概述

本文檔處理**狹義相對論**的物理框架。根據 NoiePhysicsAGENTS.md §1 的物理尺度權限層級定義，PS-LR 代表相對論效應顯著的尺度，當速度接近光速時需要使用狹義相對論。

---

## 關鍵安全與真理協議

> **CRITICAL SAFETY & TRUTH PROTOCOL:**
> 1. 遵守 AXIOMS.md 的 PT-AX17 (光速不變)、PT-AX18 (時間膨脹)
> 2. 狹義相對論是經過充分驗證的理論
> 3. 注意v > 0.1c時的相對論效應
> 4. 審計：將所有異常記錄至 PHYSICS_AUDIT_TRAIL

---

## 1. 時空結構

### 1.1 閔可夫斯基時空

$$ds^2 = -c^2dt^2 + dx^2 + dy^2 + dz^2$$

```python
class MinkowskiMetric:
    """
    閔可夫斯基度規
    """
    
    signature = [-1, 1, 1, 1]
    
    def interval(self, event1: Event, event2: Event) -> float:
        """計算時空間隔"""
        dt = event2.t - event1.t
        dx = event2.x - event1.x
        dy = event2.y - event1.y
        dz = event2.z - event1.z
        return -c**2 * dt**2 + dx**2 + dy**2 + dz**2
```

### 1.2 Lorentz 變換

$$x'^\mu = \Lambda^\mu_{\ \nu} x^\nu$$

```python
class LorentzTransformation:
    """
    Lorentz 變換
    """
    
    def lorentz_factor(self, velocity: float) -> float:
        """計算 Lorentz 因數"""
        beta = velocity / c
        return 1 / np.sqrt(1 - beta**2)
    
    def transform_coordinates(
        self,
        event: Event,
        velocity: Vector3D
    ) -> Event:
        """坐標變換"""
        gamma = self.lorentz_factor(velocity.magnitude())
        # Lorentz 變換矩陣
        pass
```

---

## 2. 相對論效應

### 2.1 時間膨脹

$$\Delta t' = \gamma \Delta t$$

```python
class TimeDilation:
    """
    時間膨脹
    """
    
    def dilated_time(
        self,
        proper_time: float,
        velocity: float
    ) -> float:
        """計算膨脹後的時間"""
        gamma = 1 / np.sqrt(1 - (velocity / c)**2)
        return proper_time * gamma
```

### 2.2 長度收縮

$$L' = \frac{L}{\gamma}$$

### 2.3 質能等價

$$E = mc^2 = \gamma m_0 c^2$$

```python
class MassEnergyEquivalence:
    """
    質能等價
    """
    
    def relativistic_energy(
        self,
        rest_mass: float,
        velocity: float
    ) -> float:
        """計算相對論能量"""
        gamma = 1 / np.sqrt(1 - (velocity / c)**2)
        return gamma * rest_mass * c**2
    
    def kinetic_energy(
        self,
        rest_mass: float,
        velocity: float
    ) -> float:
        """計算動能"""
        gamma = 1 / np.sqrt(1 - (velocity / c)**2)
        return (gamma - 1) * rest_mass * c**2
```

---

## 3. 四維表述

### 3.1 四向量

$$A^\mu = (A^0, \mathbf{A})$$

### 3.2 四動量

$$p^\mu = (E/c, \mathbf{p})$$

---

## 4. 與其他尺度的接口

### 4.1 與牛頓力學的接口

```
狹義相對論 → 牛頓力學：
- 低速極限 v << c：γ ≈ 1
- 還原為經典力學
```

---

*本文檔處理狹義相對論的物理框架。*
*狹義相對論是現代物理的基石，與實驗高度吻合。*
