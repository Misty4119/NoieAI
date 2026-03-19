# DOMAIN_MODELS/ — 無尺度知識減衰律

> **⚠️ 重要安全と真理プロトコル**：本ディレクトリは特定領域の無尺度知識減衰律を含む。知識の減衰は絶対時間ではなく、その領域の「情報代謝率」に基づく。

---

## MEDICAL_DECAY.md

### 医学的減衰領域知識減衰律

**バージョン**：v1.0 | **作成日付**：2026-03-18

---

### 領域特性

| 特性 | 説明 |
|------|------|
| **実証基盤** | 医学知識はランダム化比較試験 (RCT)、系統的レビューとメタアナリシスに基づく |
| **臨床ガイドライン** | 専門学会が发布する治療提案、定期的な更新が必要 |
| **規制フレームワーク** | 医薬品/医療機器承認、臨床試験規範 |
| **個人差** | 治療効果は人而異、精準医療の考慮が必要 |
| **情報代謝率** | 中高程度、研究発表率とガイドライン更新頻度に依存 |

---

### 減衰定数

$$\lambda^*_{\text{medical}} \approx 0.1 - 0.4$$

（具体的な値は医学知識の種類と領域に依存）

---

### 医学サブ領域減衰特徴

| サブ領域 | λ* 範囲 | 半減期（年） | トリガー要因 |
|--------|---------|--------------|----------|
| 基礎医学メカニズム | 0.05 - 0.15 | 7-15 年 | 新発見の発表 |
| 医薬品治療学 | 0.15 - 0.3 | 3-5 年 | 新薬承認、臨床試験結果 |
| 臨床ガイドライン | 0.2 - 0.35 | 2-4 年 | 系統的レビュー、ガイドライン更新 |
| 公衆衛生 | 0.1 - 0.25 | 3-7 年 | パンデミック発生、政策変更 |
| 精準医療/ゲノム学 | 0.3 - 0.5 | 1.5-2.5 年 | 新規遺伝子マーカー、検出技術 |
| 医療機器 | 0.25 - 0.45 | 2-3 年 | 技術迭代、FDA 承認 |
| 疫苗学 | 0.2 - 0.4 | 2-3 年 | 新規病原体、耐性出現 |

---

### 医学知識減衰要因

#### 1. 新規研究発表

- **PubMed 年間発表数**：約100万件の生物医学論文
- **系統的レビュー**：平均して各篇が20-50件の原研究をカバー
- **知識陳腐化臨界点**：新研究が既存のコンセンサスに挑戦した時にトリガー

#### 2. 臨床試験結果

- **大規模 RCT 発表**：既存の治療標準を覆す可能性
- **中間解析結果**：早期試験中止と臨床実践変更の可能性
- **医薬品安全警告**：副作用発見による処方変更

#### 3. 医薬品承認の動態

- **新薬上市**：既存治療法を取代する可能性
- **ジェネリック承認**：薬の近接性と処方習慣改变
- **药品撤退**：安全性または有効性の問題

#### 4. 治療法の更新

- **臨床ガイドライン改訂**：平均3-5年で大きな更新
- **新技術の導入**：CAR-T 細胞療法、遺伝子療法等
- **治療目標の進化**：症状緩和から根治へ

---

### 減衰トリガー条件

```python
MEDICAL_DECAY_TRIGGERS = [
    "新薬承認上市",
    "大規模 RCT 結果発表",
    "臨床ガイドライン更新バージョン",
    "医薬品安全警告发布",
    "治療標準変更",
    "新診断技術導入",
    "疾患分類の再定義",
    "パンデミック発生または公衆衛生危機",
    "医疗保险ポリシー変更",
    "特許満了とジェネリック競争"
]
```

---

### 具体的な半減期計算

#### 各種医学知識の減衰曲線

```python
MEDICAL_HALF_LIFE_CALCULATIONS = {
    # 基礎研究发现
    "fundamental_discovery": {
        "description": "基礎医学メカニズム発見",
        "lambda_star": 0.08,
        "half_life_years": "8-9 年",
        "examples": ["発がんメカニズム", "シグナル伝達パスウェイ"]
    },
    
    # 医薬品有効性
    "drug_efficacy": {
        "description": "特定医薬品の治療効果",
        "lambda_star": 0.2,
        "half_life_years": "3-4 年",
        "examples": ["降圧薬有効性", "抗癌剤反応率"]
    },
    
    # 臨床ガイドライン
    "clinical_guideline": {
        "description": "治療ガイドライン提案",
        "lambda_star": 0.25,
        "half_life_years": "2.5-3 年",
        "examples": ["糖尿病治療ガイドライン", "心臓病一次予防"]
    },
    
    # 精準医療マーカー
    "precision_biomarker": {
        "description": "バイオマーカーと遺伝子検査",
        "lambda_star": 0.4,
        "half_life_years": "1.5-2 年",
        "examples": ["腫瘍変異検出", "薬物ゲノム学"]
    },
    
    # 公衆衛生提案
    "public_health": {
        "description": "公衆衛生政策と提案",
        "lambda_star": 0.15,
        "half_life_years": "4-5 年",
        "examples": ["ワクチン接種提案", "|CAN 頻度"]
    },
    
    # 医療機器技術
    "medical_device": {
        "description": "医療機器と技術",
        "lambda_star": 0.35,
        "half_life_years": "2 年",
        "examples": ["インプラント", "診断機器"]
    }
}

# 減衰公式応用
def calculate_medical_validity(initial_validity, years, lambda_star):
    """
    医学知識が t 年経過後の有効性を計算
    
    Parameters:
    - initial_validity: 原始有効性 (0-1)
    - years: 経過年数
    - lambda_star: 減衰定数
    
    Returns:
    - current_validity: 現在有効性
    """
    current_validity = initial_validity * math.exp(-lambda_star * years)
    return max(current_validity, 0.01)  # 最低有効性閾値を設定
```

#### 実証医学証拠レベルと減衰

```python
EVIDENCE_LEVEL_DECAY = {
    "Level_A": {
        "description": "複数の大規模 RCT の系統的レビュー/メタアナリシス",
        "base_reliability": 0.95,
        "lambda_star": 0.1,
        "half_life": "7 年",
        "decay_pattern": "緩やかな減衰、重大な新証拠でのみ显著に変化"
    },
    
    "Level_B": {
        "description": "単一の高品質 RCT または複数の高品質コホート研究",
        "base_reliability": 0.85,
        "lambda_star": 0.2,
        "half_life": "3.5 年",
        "decay_pattern": "中等減衰、新しい試験結果が結論を変更する可能性"
    },
    
    "Level_C": {
        "description": "専門家コンセンサス/患者対照研究",
        "base_reliability": 0.7,
        "lambda_star": 0.35,
        "half_life": "2 年",
        "decay_pattern": "较快な減衰、より高质量な証拠での検証が必要"
    },
    
    "Level_D": {
        "description": "症例報告/専門家意見",
        "base_reliability": 0.5,
        "lambda_star": 0.5,
        "half_life": "1.4 年",
        "decay_pattern": "急速な減衰、臨床適用性は限定的"
    }
}
```

---

### 医学知識検証と更迭

**重大な医学的進歩**：

| 発見/発展 | 時間 | 重要性 | 既存知識への影響 |
|-----------|------|--------|------------------|
| **mRNA 疫苗技術最適化** | 2025-2026 | 重大 | 疫苗開発プラットフォームが成熟、速度と安全性が向上 |
| **アルツハイマー病抗体薬** | 2025 | 重要 | Lecanemab/Donanemab が承認、治療パラダイムが変化 |
| **CRISPR 遺伝子編集治療** | 2025-2026 | 革命的 | Casgevy が承認、体内遺伝子編集药的初承認 |
| **肥胖症药物突破** | 2025 | 重大 | GLP-1 类似物（semaglutide, tirzepatide）が治療格局を変更 |
| **AI 補助画像診断** | 2025-2026 | 重要 | 深層学習により癌篩検精度が向上 |
| **腸管微生物叢療法** | 2025 | 継続 | FMT とプロバイオティクス制剂が臨床試験に進む |
| **細胞療法 CAR-T 拡大** | 2025-2026 | 重要 | 血液腫瘍から固形腫瘍への拡大 |

**新規減衰トリガー条件**：

```python
MEDICAL_DECAY_TRIGGERS = [
    # 既存のトリガー条件
    "新薬承認上市",
    "大規模 RCT 結果発表",
    "臨床ガイドライン更新バージョン",
    # 新規追加
    "GLP-1 系減量薬承認",
    "アルツハイマー病抗体薬の実世界データ",
    "遺伝子編集治療初例承認",
    "AI 診断システム規制承認",
    "COVID-19 後遺症研究結論",
    "抗生素耐性監視データ更新",
    "新型冠状病毒株進化追跡"
]
```

---

### 他の領域との相互参照

#### 医学 × 科学

- **基礎科学発見** → 医薬品ターゲット同定 → 臨床試験 → 承認上市
- **λ* 相互作用**：基礎科学の λ* (≈0) が医学応用の λ* (0.1-0.4) に影響

#### 医学 × 技術

- **医療機器** → AI 診断 → データ駆動治療
- **λ* 相互作用**：技術の高い λ* (0.3-0.7) が医学応用の減衰を加速

#### 医学 × 法律

- **医薬品規制** → 特許保護 → ジェネリック上市
- **λ* 相互作用**：法律の λ* (≈中程度-低) が医薬品ライフサイクルに影響

#### 医学 × ニュース

- **健康ニュース** → 公众認識 → 医療意思決定
- **λ* 相互作用**：ニュースの高い λ* が医学知識の誤解传播を引き起こす可能性

---

### 計算公式

```python
FUNCTION ComputeMedicalValidity(claim, current_year, original_year, evidence_level):
    years_elapsed = current_year - original_year
    lambda_star = EVIDENCE_LEVEL_DECAY[evidence_level]["lambda_star"]
    base_reliability = EVIDENCE_LEVEL_DECAY[evidence_level]["base_reliability"]
    
    decay = exp(-lambda_star * years_elapsed)
    current_validity = base_reliability * decay
    
    # 新証拠の蓄積効果を考慮
    new_evidence_factor = min(1.0 + 0.1 * CountNewRCTs(claim.topic, original_year, current_year), 1.5)
    
    RETURN min(current_validity * new_evidence_factor, 1.0)
```

---

### 減衰公式

$$Validity(K, t) = V_0 \cdot e^{-\lambda^* \cdot t}$$

ここで $t$ は経過年数。

### 無尺度半減期

$$\nu_{1/2} = \frac{\ln(2)}{\lambda^*}$$

「領域状態が何度翻转した後、知識有効性が半分になるか」を表す。

---

### 引用と参照

- 参照：**SCIENTIFIC_DECAY.md** — 基礎科学発見
- 参照：**TECH_DECAY.md** — 医療技術と AI 診断
- 参照：**LEGAL_DECAY.md** — 医薬品規制と特許
- 参照：**NEWS_DECAY.md** — 健康ニュース传播

---

**バージョン履歴**：
- v1.0 (2026-03-18): 初期バージョン、医学領域知識減衰モデルを構築
