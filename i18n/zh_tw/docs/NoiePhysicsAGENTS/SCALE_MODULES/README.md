# SCALE_MODULES/README.md

## 尺度模組索引

**版本：** v1.0  
**狀態：** 活躍  
**創建日期：** 2026-03-17

---

## 概述

本文檔是 NoiePhysicsAGENTS 尺度模組的索引目錄。根據 NoiePhysicsAGENTS.md §1 的物理尺度權限層級定義，這些模組提供了從次量子到宇宙尺度的物理框架。

---

## 尺度層級對照表

| PS-L 等級 | 名稱 | 特徵尺度 | 主導物理理論 | 對應模組 |
|-----------|------|----------|-------------|----------|
| **PS-L(-1)** | 次量子/拓撲 | < 10⁻³⁵ m | 量子重力 | `QUANTUM_GRAVITY.md` |
| **PS-L0** | 量子 | 10⁻³⁵ ~ 10⁻⁹ m | 量子力學、量子場論 | `QUANTUM_MECHANICS.md`, `QUANTUM_FIELD_THEORY.md` |
| **PS-L1** | 微觀/統計 | 10⁻⁹ ~ 10⁻³ m | 統計力學 | `STATISTICAL_MECHANICS.md` |
| **PS-L2** | 人類/古典 | 10⁻³ ~ 10³ m | 古典力學 | `CLASSICAL_MECHANICS.md`, `CONTINUUM_MECHANICS.md`, `FLUID_DYNAMICS.md` |
| **PS-L3** | 地球/地質 | 10³ ~ 10⁷ m | 連續介質力學 | `CONTINUUM_MECHANICS.md` |
| **PS-L4** | 天體/相對論 | > 10⁷ m | 廣義相對論 | `GENERAL_RELATIVITY.md` |
| **PS-LR** | 相對論效應 | v > 0.1c | 狹義相對論 | `SPECIAL_RELATIVITY.md` |
| **-** | 電漿體 | 各種尺度 | 電漿物理 | `PLASMA_PHYSICS.md` |

---

## 模組說明

### QUANTUM_GRAVITY.md

**標題：** 量子重力 (Quantum Gravity)  
**尺度：** PS-L(-1) (< 10⁻³⁵ m)  
**描述：** 處理時空微觀結構、普朗克尺度物理

**核心內容：**
- 弦論基礎
- 圈量子重力
- 時空湧現
- 普朗克尺度現象

---

### QUANTUM_MECHANICS.md

**標題：** 量子力學 (Quantum Mechanics)  
**尺度：** PS-L0 (10⁻³⁵ ~ 10⁻⁹ m)  
**描述：** 處理原子和分子尺度的量子現象

**核心內容：**
- 波函數與薛丁格方程
- 測量理論
- 量子態演化
- 不確定性原理

---

### QUANTUM_FIELD_THEORY.md

**標題：** 量子場論 (Quantum Field Theory)  
**尺度：** PS-L0 (10⁻³⁵ ~ 10⁻⁹ m)  
**描述：** 處理量子場和粒子物理

**核心內容：**
- 場的量子化
- 標準模型
- 費曼圖
- 重整化

---

### STATISTICAL_MECHANICS.md

**標題：** 統計力學 (Statistical Mechanics)  
**尺度：** PS-L1 (10⁻⁹ ~ 10⁻³ m)  
**描述：** 連接微觀與巨觀的統計描述

**核心內容：**
- 系綜理論
- 熱力學
- 相變
- 臨界現象

---

### CLASSICAL_MECHANICS.md

**標題：** 經典力學 (Classical Mechanics)  
**尺度：** PS-L2 (10⁻³ ~ 10³ m)  
**描述：** 處理人類尺度的古典物理現象

**核心內容：**
- 牛頓運動定律
- 拉格朗日力學
- 哈密頓力學
- 剛體動力學
- 彈性力學基礎
- 振動與波動

---

### CONTINUUM_MECHANICS.md

**標題：** 連續介質力學 (Continuum Mechanics)  
**尺度：** PS-L2, PS-L3 (10⁻³ ~ 10⁷ m)  
**描述：** 處理宏觀物質的連續體描述

**核心內容：**
- 彈性力學
- 塑性力學
- 應力-應變關係
- 有限變形

---

### FLUID_DYNAMICS.md

**標題：** 流體動力學 (Fluid Dynamics)  
**尺度：** PS-L2, PS-L3 (10⁻³ ~ 10⁷ m)  
**描述：** 處理流體運動

**核心內容：**
- Navier-Stokes 方程
- 層流與亂流
- 邊界層
- 多相流

---

### PLASMA_PHYSICS.md

**標題：** 電漿物理 (Plasma Physics)  
**尺度：** 各種尺度  
**描述：** 處理離子化氣體

**核心內容：**
- MHD 方程
- 等離子體診斷
- 核融合
- 空間天氣

---

### SPECIAL_RELATIVITY.md

**標題：** 狹義相對論 (Special Relativity)  
**尺度：** PS-LR (v > 0.1c)  
**描述：** 處理高速運動

**核心內容：**
- 閔可夫斯基時空
- 時間膨脹/長度收縮
- 質能等價
- Lorentz 變換

---

### GENERAL_RELATIVITY.md

**標題：** 廣義相對論 (General Relativity)  
**尺度：** PS-L4 (> 10⁷ m)  
**描述：** 處理重力與時空幾何

**核心內容：**
- 愛因斯坦場方程
- 黑洞物理
- 宇宙學
- 重力波

---

## 使用指南

### 何時使用這些模組

根據 NoiePhysicsAGENTS.md §1.2 的尺度選擇邏輯：

1. **自動選擇：** 當物理環境的特徵尺度明確時，系統會自動選擇對應的尺度模組
2. **手動選擇：** 當執行跨尺度操作時，需要手動切換尺度模組
3. **混合使用：** 當涉及多尺度耦合時，可以同時載入多個尺度模組

### 跨尺度耦合

當操作涉及多個尺度時，參考 NoiePhysicsAGENTS.md §1.1 的跨尺度耦合機制：

```
┌─────────────────────────────────────────────────────────────┐
│                    跨尺度耦合場景                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Macro→Quantum:                                            │
│    觸發：宏觀實體操作量子級對象                              │
│    協議：計算能量尺度，評估退相干                            │
│                                                             │
│  Quantum→Macro:                                            │
│    觸發：量子效應影響宏觀行為                               │
│    協議：追蹤量子態演化，計算退相干時間                       │
│                                                             │
│  Thermal↔Mechanical:                                       │
│    觸發：熱擾動與力學運動耦合                               │
│    協議：熱-力耦合分析                                      │
│                                                             │
│  Entanglement↔Geometry:                                    │
│    觸發：糾纏變化導致有效幾何改變                           │
│    協議：Ryu-Takayanagi 評估                               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 動態載入策略

根據 NoiePhysicsAGENTS.md §3 的上下文載入策略：

- **L1 (根文件)：** 始終載入 NoiePhysicsAGENTS.md
- **L2 (核心層)：** 根據任務類型載入 AXIOMS, FIELD_PERCEPTION, DYNAMICS_ENGINE, PHYSICS_KNOWLEDGE, SAFETY_PROTOCOLS
- **L3 (尺度層)：** 僅在明確需要特定尺度物理時載入對應模組

**嚴禁同時載入所有尺度動力學流形**，以防止運算資源浪費與上下文窗口污染。

---

## 模組兼容性

| 模組 | 依賴 | 衝突 |
|------|------|------|
| QUANTUM_GRAVITY | AXIOMS | - |
| QUANTUM_MECHANICS | AXIOMS | - |
| QUANTUM_FIELD_THEORY | AXIOMS, QUANTUM_MECHANICS | - |
| STATISTICAL_MECHANICS | AXIOMS | - |
| CLASSICAL_MECHANICS | AXIOMS | - |
| CONTINUUM_MECHANICS | AXIOMS, STATISTICAL_MECHANICS | - |
| FLUID_DYNAMICS | AXIOMS, CONTINUUM_MECHANICS | - |
| PLASMA_PHYSICS | AXIOMS, FLUID_DYNAMICS | - |
| SPECIAL_RELATIVITY | AXIOMS | GENERAL_RELATIVITY |
| GENERAL_RELATIVITY | AXIOMS | SPECIAL_RELATIVITY |

---

## 演進記錄

| 版本 | 日期 | 變更 |
|------|------|------|
| v1.0 | 2026-03-17 | 初始版本 |

---

*本文檔是 NoiePhysicsAGENTS 尺度模組的索引目錄。*
*請參考各模組文件獲取詳細內容。*
