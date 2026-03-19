# DOMAIN_MODELS/ — 無尺度知識衰減律

> **⚠️ 關鍵安全與真理協議**：本目錄包含特定領域的無尺度知識衰減律。知識的衰減不基於絕對時間，而是基於該領域的「資訊代謝率」。

---

## MEDICAL_DECAY.md

### 醫學領域知識衰減律

**版本**：v1.0 | **建立日期**：2026-03-18

---

### 領域特性

| 特性 | 描述 |
|------|------|
| **實證基礎** | 醫學知識基於隨機對照試驗 (RCT)、系統性回顧與統合分析 |
| **臨床指引** | 專業學會發布的治療建議，需定期更新 |
| **監管框架** | 藥物/醫療器材審批、臨床試驗規範 |
| **個體差異** | 治療效果因人而異，需精準醫學考量 |
| **資訊代謝率** | 中高程度，取決於研究發表率與指南更新頻率 |

---

### 衰減常數

$$\lambda^*_{\text{medical}} \approx 0.1 - 0.4$$

（具體取決於醫學知識類型與領域）

---

### 醫學子領域衰減特徵

| 子領域 | λ* 範圍 | 半衰期（年） | 觸發因素 |
|--------|---------|--------------|----------|
| 基礎醫學機制 | 0.05 - 0.15 | 7-15 年 | 新發現發表 |
| 藥物治療學 | 0.15 - 0.3 | 3-5 年 | 新藥審批、臨床試驗結果 |
| 臨床指引 | 0.2 - 0.35 | 2-4 年 | 系統性回視、指南更新 |
| 公共衛生 | 0.1 - 0.25 | 3-7 年 | 疫情爆發、政策變更 |
| 精準醫學/基因體 | 0.3 - 0.5 | 1.5-2.5 年 | 新基因標記、檢測技術 |
| 醫療器材 | 0.25 - 0.45 | 2-3 年 | 技術迭代、FDA 審批 |
| 疫苗學 | 0.2 - 0.4 | 2-3 年 | 新病原體、出現抗藥性 |

---

### 醫學知識衰減因素

#### 1. 新研究發表

- **PubMed 每年發表**：約 100 萬篇生物醫學論文
- **系統性回顧**：平均每篇涵蓋 20-50 篇原始研究
- **知識過時臨界點**：當新研究挑戰現有共識時觸發

#### 2. 臨床試驗結果

- **大規模 RCT 發表**：可能推翻現有治療標準
- **中期分析結果**：可能提前終止試驗並改變臨床實踐
- **藥物安全警訊**：不良反應發現導致處方變更

#### 3. 藥物審批動態

- **新藥上市**：可能取代現有治療方案
- **仿製藥核准**：改變藥物可近性與處方習慣
- **藥物下市**：安全性或有效性問題

#### 4. 治療方式更新

- **臨床指引修訂**：平均 3-5 年一次大更新
- **新技術引入**：如 CAR-T 細胞治療、基因治療
- **治療目標演變**：從緩解症狀到根治疾病

---

### 衰減觸發條件

```python
MEDICAL_DECAY_TRIGGERS = [
    "新藥核准上市",
    "大規模 RCT 結果發表",
    "臨床指引更新版本",
    "藥物安全警訊發布",
    "治療標準改變",
    "新診斷技術引入",
    "疾病分類重新定義",
    "疫情爆發或公共衛生危機",
    "醫療保險政策變更",
    "專利到期與仿製藥競爭"
]
```

---

### 具體半衰期計算

#### 不同類型醫學知識的衰減曲線

```python
MEDICAL_HALF_LIFE_CALCULATIONS = {
    # 基礎研究發現
    "fundamental_discovery": {
        "description": "基礎醫學機制發現",
        "lambda_star": 0.08,
        "half_life_years": "8-9 年",
        "examples": ["致癌機制", "訊號傳導路徑"]
    },
    
    # 藥物療效
    "drug_efficacy": {
        "description": "特定藥物治療效果",
        "lambda_star": 0.2,
        "half_life_years": "3-4 年",
        "examples": ["降壓藥療效", "抗癌藥反應率"]
    },
    
    # 臨床指引
    "clinical_guideline": {
        "description": "治療指引建議",
        "lambda_star": 0.25,
        "half_life_years": "2.5-3 年",
        "examples": ["糖尿病治療指引", "心臟病初級預防"]
    },
    
    # 精準醫學標記
    "precision_biomarker": {
        "description": "生物標記與基因檢測",
        "lambda_star": 0.4,
        "half_life_years": "1.5-2 年",
        "examples": ["腫瘤突變檢測", "藥物基因體學"]
    },
    
    # 公共衛生建議
    "public_health": {
        "description": "公共衛生政策與建議",
        "lambda_star": 0.15,
        "half_life_years": "4-5 年",
        "examples": ["疫苗施打建議", "篩檢頻率"]
    },
    
    # 醫療器材技術
    "medical_device": {
        "description": "醫療器材與技術",
        "lambda_star": 0.35,
        "half_life_years": "2 年",
        "examples": ["植入物", "診斷設備"]
    }
}

# 衰減公式應用
def calculate_medical_validity(initial_validity, years, lambda_star):
    """
    計算醫學知識在經過 t 年後的有效性
    
    Parameters:
    - initial_validity: 原始有效性 (0-1)
    - years: 經過的年數
    - lambda_star: 衰減常數
    
    Returns:
    - current_validity: 當前有效性
    """
    current_validity = initial_validity * math.exp(-lambda_star * years)
    return max(current_validity, 0.01)  # 設定最低有效性門檻
```

#### 實證醫學證據等級與衰減

```python
EVIDENCE_LEVEL_DECAY = {
    "Level_A": {
        "description": "多個大規模 RCT 的系統性回顧/統合分析",
        "base_reliability": 0.95,
        "lambda_star": 0.1,
        "half_life": "7 年",
        "decay_pattern": "緩慢衰減，需重大新證據才會顯著改變"
    },
    
    "Level_B": {
        "description": "單一高品質 RCT 或多個高品質隊列研究",
        "base_reliability": 0.85,
        "lambda_star": 0.2,
        "half_life": "3.5 年",
        "decay_pattern": "中等衰減，新試驗結果可能改變結論"
    },
    
    "Level_C": {
        "description": "專家共識/病例對照研究",
        "base_reliability": 0.7,
        "lambda_star": 0.35,
        "half_life": "2 年",
        "decay_pattern": "較快衰減，需更高質量證據驗證"
    },
    
    "Level_D": {
        "description": "個案報告/專家意見",
        "base_reliability": 0.5,
        "lambda_star": 0.5,
        "half_life": "1.4 年",
        "decay_pattern": "快速衰減，臨床適用性有限"
    }
}
```

---

### 醫學知識驗證與更迭

**重大醫學進展**：

| 發現/發展 | 時間 | 重要性 | 對現有知識的影響 |
|-----------|------|--------|------------------|
| **mRNA 疫苗技術優化** | 2025-2026 | 重大 | 疫苗開發平台成熟，速度與安全性提升 |
| **阿茲海默症抗體藥物** | 2025 | 重要 | Lecanemab/Donanemab 獲核准，改變治療範式 |
| **CRISPR 基因編輯治療** | 2025-2026 | 革命性 | Casgevy 獲批，首個體內基因編輯藥物 |
| **肥胖症藥物突破** | 2025 | 重大 | GLP-1 類似物（semaglutide, tirzepatide）改變治療格局 |
| **AI 輔助影像診斷** | 2025-2026 | 重要 | 深度學習提高癌症篩檢準確率 |
| **腸道微生物組療法** | 2025 | 持續 | FMT 與益生菌製劑進入臨床試驗 |
| **細胞療法 CAR-T 擴展** | 2025-2026 | 重要 | 從血液腫瘤擴展至實體瘤 |

**新增衰減觸發條件**：

```python
MEDICAL_DECAY_TRIGGERS = [
    # 原有觸發條件
    "新藥核准上市",
    "大規模 RCT 結果發表",
    "臨床指引更新版本",
    # 新增
    "GLP-1 類減重藥物核准",
    "阿茲海默症抗體藥物真實世界數據",
    "基因編輯治療首例獲批",
    "AI 診斷系統法規核准",
    "COVID-19 長期後遺症研究結論",
    "抗生素抗藥性監測數據更新",
    "新型冠狀病毒株演變追蹤"
]
```

---

### 與其他領域的交叉引用

#### 醫學 × 科學

- **基礎科學發現** → 藥物靶點識別 → 臨床試驗 → 核准上市
- **λ* 互動**：基礎科學的 λ* (≈0) 影響醫學應用的 λ* (0.1-0.4)

#### 醫學 × 技術

- **醫療器材** → AI 診斷 → 數據驅動治療
- **λ* 互動**：技術的高 λ* (0.3-0.7) 加速醫學應用的衰減

#### 醫學 × 法律

- **藥物法規** → 專利保護 → 仿製藥上市
- **λ* 互動**：法律的 λ* (≈medium-low) 影響藥物生命週期

#### 醫學 × 新聞

- **健康新聞** → 公眾認知 → 醫療決策
- **λ* 互動**：新聞的高 λ* 可能導致醫學知識的誤解傳播

---

### 計算公式

```python
FUNCTION ComputeMedicalValidity(claim, current_year, original_year, evidence_level):
    years_elapsed = current_year - original_year
    lambda_star = EVIDENCE_LEVEL_DECAY[evidence_level]["lambda_star"]
    base_reliability = EVIDENCE_LEVEL_DECAY[evidence_level]["base_reliability"]
    
    decay = exp(-lambda_star * years_elapsed)
    current_validity = base_reliability * decay
    
    # 考慮新證據的累積效應
    new_evidence_factor = min(1.0 + 0.1 * CountNewRCTs(claim.topic, original_year, current_year), 1.5)
    
    RETURN min(current_validity * new_evidence_factor, 1.0)
```

---

### 衰減公式

$$Validity(K, t) = V_0 \cdot e^{-\lambda^* \cdot t}$$

其中 $t$ 為經過的年數。

### 無尺度半衰期

$$\nu_{1/2} = \frac{\ln(2)}{\lambda^*}$$

表示「經過多少次領域狀態翻轉後，知識有效性減半」。

---

### 引用與參照

- 參照：**SCIENTIFIC_DECAY.md** — 基礎科學發現
- 參照：**TECH_DECAY.md** — 醫療技術與 AI 診斷
- 參照：**LEGAL_DECAY.md** — 藥物法規與專利
- 參照：**NEWS_DECAY.md** — 健康新聞傳播

---

**版本歷史**：
- v1.0 (2026-03-18): 初始版本，建立醫學領域知識衰減模型
