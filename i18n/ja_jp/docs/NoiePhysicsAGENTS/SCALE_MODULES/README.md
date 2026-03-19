# SCALE_MODULES/README.md

## スケールモジュール索引

**バージョン:** v1.0  
**状態:** アクティブ  
**作成日:** 2026-03-17

---

## 概要

本文書はNoiePhysicsAGENTSスケールモジュールの索引ディレクトリである。NoiePhysicsAGENTS.md §1の物理スケール権限レベル定義に従い、これらのモジュールは亜量子から宇宙スケールまでの物理フレームワークを提供する。

---

## スケールレベル対応表

| PS-L レベル | 名称 | 特徴スケール | 主導物理理論 | 対応モジュール |
|------------|------|----------|-------------|----------|
| **PS-L(-1)** | 亜量子/位相的 | < 10⁻³⁵ m | 量子重力 | `QUANTUM_GRAVITY.md` |
| **PS-L0** | 量子 | 10⁻³⁵ ~ 10⁻⁹ m | 量子力学、量子場理論 | `QUANTUM_MECHANICS.md`, `QUANTUM_FIELD_THEORY.md` |
| **PS-L1** | 微視的/統計的 | 10⁻⁹ ~ 10⁻³ m | 統計力学 | `STATISTICAL_MECHANICS.md` |
| **PS-L2** | 人間/古典的 | 10⁻³ ~ 10³ m | 古典力学 | `CLASSICAL_MECHANICS.md`, `CONTINUUM_MECHANICS.md`, `FLUID_DYNAMICS.md` |
| **PS-L3** | 地球/地質的 | 10³ ~ 10⁷ m | 連続体力学 | `CONTINUUM_MECHANICS.md` |
| **PS-L4** | 天体/相対論的 | > 10⁷ m | 一般相対性理論 | `GENERAL_RELATIVITY.md` |
| **PS-LR** | 相対論的効果 | v > 0.1c | 特殊相対性理論 | `SPECIAL_RELATIVITY.md` |
| **-** | プラズマ体 | 各種スケール | プラズマ物理 | `PLASMA_PHYSICS.md` |

---

## モジュール説明

### QUANTUM_GRAVITY.md

**タイトル：** 量子重力 (Quantum Gravity)  
**スケール：** PS-L(-1) (< 10⁻³⁵ m)  
**記述：** 時空の微視的構造、プランクスケール物理を処理

**コア内容：**
- 弦理論基礎
- ループ量子重力
- 時空湧現
- プランクスケール現象

---

### QUANTUM_MECHANICS.md

**タイトル：** 量子力学 (Quantum Mechanics)  
**スケール：** PS-L0 (10⁻³⁵ ~ 10⁻⁹ m)  
**記述：** 原子・分子スケールの量子現象を処理

**コア内容：**
- 波動関数とシュレディンガー方程式
- 測定理論
- 量子状態の時間発展
- 不確定性原理

---

### QUANTUM_FIELD_THEORY.md

**タイトル：** 量子場理論 (Quantum Field Theory)  
**スケール：** PS-L0 (10⁻³⁵ ~ 10⁻⁹ m)  
**記述：** 量子場と粒子物理を処理

**コア内容：**
- 場の量子化
- 標準模型
- ファインマン図
- 再正規化

---

### STATISTICAL_MECHANICS.md

**タイトル：** 統計力学 (Statistical Mechanics)  
**スケール：** PS-L1 (10⁻⁹ ~ 10⁻³ m)  
**記述：** 微視的と巨視的を繋ぐ統計的記述

**コア内容：**
- 集合理論
- 熱力学
- 相転移
- 臨界現象

---

### CLASSICAL_MECHANICS.md

**タイトル：** 古典力学 (Classical Mechanics)  
**スケール：** PS-L2 (10⁻³ ~ 10³ m)  
**記述：** 人間スケールの古典物理現象を処理

**コア内容：**
- ニュートンの運動法則
- ラグランダン力学
- ハミルトン力学
- 剛体動力学
- 弾性力学基礎
- 振動と波動

---

### CONTINUUM_MECHANICS.md

**タイトル：** 連続体力学 (Continuum Mechanics)  
**スケール：** PS-L2, PS-L3 (10⁻³ ~ 10⁷ m)  
**記述：** 巨視的物質の連続体記述を処理

**コア内容：**
- 弾性力学
- 塑性力学
- 応力-ひずみ関係
- 有限変形

---

### FLUID_DYNAMICS.md

**タイトル：** 流体力学 (Fluid Dynamics)  
**スケール：** PS-L2, PS-L3 (10⁻³ ~ 10⁷ m)  
**記述：** 流体運動を処理

**コア内容：**
- Navier-Stokes方程式
- 層流と乱流
- 境界層
- 多相流

---

### PLASMA_PHYSICS.md

**タイトル：** プラズマ物理 (Plasma Physics)  
**スケール：** 各種スケール  
**記述：** 電離気体を処理

**コア内容：**
- MHD方程式
- プラズマ診断
- 核融合
- 宇宙天気

---

### SPECIAL_RELATIVITY.md

**タイトル：** 特殊相対性理論 (Special Relativity)  
**スケール：** PS-LR (v > 0.1c)  
**記述：** 高速運動を処理

**コア内容：**
- ミンコフスキー時空
- 時間遅延/長さ収縮
- 質量エネルギー等価性
- ローレンツ変換

---

### GENERAL_RELATIVITY.md

**タイトル：** 一般相対性理論 (General Relativity)  
**スケール：** PS-L4 (> 10⁷ m)  
**記述：** 重力と時空幾何学を処理

**コア内容：**
- アinstein場方程式
- ブラックホール物理
- 宇宙論
- 重力波

---

## 使用ガイド

### いつこれらのモジュールを使用するか

NoiePhysicsAGENTS.md §1.2のスケール選択論理に従い：

1. **自動選択：** 物理環境の特徵スケールが明確な時、系は自動的に対応するスケールモジュールを選択する
2. **手動選択：** 跨スケール操作を実行する時、スケールモジュールを手動で切り替える必要がある
3. **混合使用：** 複数スケール結合を含む場合、複数のスケールモジュールを同時にロードできる

### 跨スケール結合

操作が複数のスケールを含む時、NoiePhysicsAGENTS.md §1.1の跨スケール結合メカニズムを参照：

```
┌─────────────────────────────────────────────────────────────┐
│                    跨スケール結合シナリオ                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Macro→Quantum:                                           │
│    トリガー：巨視的実体が量子レベルオブジェクトを操作          │
│    プロトコル：エネルギースケールを計算、量子デコヒーレンスを評価 │
│                                                             │
│  Quantum→Macro:                                           │
│    トリガー：量子効果が巨視的挙動に影響                      │
│    プロトコル：量子状態時間発展を追跡、量子デコヒーレンス時間を計算│
│                                                             │
│  Thermal↔Mechanical:                                       │
│    トリガー：熱撹乱と力学的運動が結合                        │
│    プロトコル：熱-力結合解析                                 │
│                                                             │
│  Entanglement↔Geometry:                                    │
│    トリガー：エンタングルメント変化が有効幾何学的変化をもたら   │
│    プロトコル：Ryu-Takayanagi評価                           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 動的ロード戦略

NoiePhysicsAGENTS.md §3のコンテキストロード戦略に従い：

- **L1 (ルートファイル)：** 常時NoiePhysicsAGENTS.mdをロード
- **L2 (コアレベル)：** タスクタイプに従いAXIOMS, FIELD_PERCEPTION, DYNAMICS_ENGINE, PHYSICS_KNOWLEDGE, SAFETY_PROTOCOLSをロード
- **L3 (スケールレベル)：** 特定のスケール物理を明示的に必要とする時のみ対応モジュールをロード

**全スケール動力学多様体を同時にロードすることは厳禁**であり、演算リソースの浪費とコンテキストウィンドウの汚染を防止するためである。

---

## モジュール互換性

| モジュール | 依存 | 競合 |
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

## 進化記録

| バージョン | 日付 | 変更 |
|------|------|------|
| v1.0 | 2026-03-17 | 初期バージョン |

---

*本文書はNoiePhysicsAGENTSスケールモジュールの索引ディレクトリである。*
*詳細な内容は各モジュールの文書を参照されたい。*
