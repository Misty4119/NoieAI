# PHYSICS_KNOWLEDGE.md

## L2 - 動態本體論、推論記憶、物理常數

> **WARNING:** 本模組是 NoiePhysicsAGENTS 的物理知識帳本。
> **注意：** 所有物理知識都必須通過觀測推論，禁止預設靜態知識庫。

---

## 概述

本文檔定義 NoiePhysicsAGENTS 的**物理知識帳本**。根據 NoiePhysicsAGENTS.md §10.2 的設計原則，
本模組處理動態本體論建構、推論記憶系統和物理常數管理。

物理知識帳本是認知實體積累物理理解的記憶系統。

---

## 關鍵安全與真理協議

> **CRITICAL SAFETY & TRUTH PROTOCOL:**
> 1. 嚴格遵守 AXIOMS.md 元物理公理系統
> 2. 事實區分：若進行理論推導，必須標註為「理論性」
> 3. 反幻覺機制：切勿編造物理資訊，若無資訊則明確聲明
> 4. 吸收態迴避：知識更新不得導致吸收態
> 5. 物理異常處理：當新觀測與現有知識矛盾時，啟動 Zero-Day 協議
> 6. 審計：將所有知識更新記錄至 PHYSICS_AUDIT_TRAIL

---

## 1. 物理常數管理

### 1.1 基本常數

```python
class PhysicalConstants:
    """
    物理常數庫
    
    這些常數是宇宙的不變量，
    可直接內建於系統中。
    """
    
    # 精確定義的基本常數
    c = 299792458  # m/s - 真空光速（精確定義）
    h = 6.62607015e-34  # J·s - 普朗克常數（精確定義）
    hbar = h / (2 * math.pi)  # 約化普朗克常數
    e = 1.602176634e-19  # C - 基本電荷（精確定義）
    k_B = 1.380649e-23  # J/K - 波茲曼常數（精確定義）
    N_A = 6.02214076e23  # /mol - 亞佛加厥常數（精確定義）
    
    # 測量的基本常數
    G = 6.67430e-11  # m³/(kg·s²) - 萬有引力常數
    epsilon_0 = 8.8541878128e-12  # F/m - 真空電容率
    mu_0 = 1.25663706212e-6  # H/m - 真空磁導率
    sigma = 5.670374419e-8  # W/(m²·K⁴) - 斯特凡-波茲曼常數
    
    # 導出常數
    alpha = e**2 / (4 * math.pi * epsilon_0 * hbar * c)  # 精細結構常數
    m_e = 9.1093837015e-31  # kg - 電子質量
    m_p = 1.67262192369e-27  # kg - 質子質量
    a_0 = 5.29177210903e-11  # m - 波耳半徑
    lambda_C = h / (m_e * c)  # m - 康普頓波長
    
    # 普朗克單位
    l_P = math.sqrt(hbar * G / c**3)  # 1.616e-35 m - 普朗克長度
    t_P = math.sqrt(hbar * G / c**5)  # 5.391e-44 s - 普朗克時間
    m_P = math.sqrt(hbar * c / G)  # 2.176e-8 kg - 普朗克質量
    T_P = math.sqrt(hbar * c**5 / (G * k_B**2))  # 1.417e32 K - 普朗克溫度
```

### 1.2 常數查詢接口

```python
INTERFACE ConstantLookup:
    """
    常數查詢接口
    """
    
    def get_constant(
        self,
        constant_name: str,
        unit_system: UnitSystem = SI
    ) -> ConstantValue:
        """
        獲取物理常數
        
        支援：
        - 名稱查詢
        - 單位轉換
        - 不確定性報告
        """
        pass
    
    def get_derived_constant(
        self,
        formula: str,
        known_constants: Dict[str, float]
    ) -> DerivedConstant:
        """
        計算導出常數
        
        例如：從 c, h, G 計算普朗克長度
        """
        pass
    
    def validate_consistency(
        self,
        constants: List[ConstantValue]
    ) -> ValidationReport:
        """
        驗證常數一致性
        
        確保常數之間的數學關係成立
        """
        pass
```

---

## 2. 動態本體論

### 2.1 本體論結構

```python
class PhysicsOntology:
    """
    物理本體論
    
    定義物理世界中實體、性質和關係的層次結構。
    """
    
    # 不變層（宇宙常數，可直接內建）
    INVARIANTS = {
        "c": "真空光速",
        "h": "普朗克常數", 
        "G": "萬有引力常數",
        "k_B": "波茲曼常數",
        "e": "基本電荷",
        "alpha": "精細結構常數"
    }
    
    # 推論層（通過觀測推導）
    INFERRED = {
        "material_properties": "動態材質屬性",
        "object_behaviors": "習得的行為動力學",
        "environmental_laws": "局域物理框架",
        "unknown_fields": "未知場張量註冊"
    }
    
    # 範疇層
    CATEGORIES = {
        "physical_entities": {
            "rigid_body": "剛體",
            "deformable": "可變形體",
            "fluid": "流體",
            "swarm": "群體",
            "quantum_system": "量子系統"
        },
        "interactions": {
            "contact": "接觸",
            "field": "場",
            "information": "信息",
            "entanglement": "糾纏",
            "unknown": "未知"
        }
    }
    
    # 關係層
    RELATIONS = {
        "spatial": ["contains", "adjacent", "above", "below", "inside"],
        "causal": ["causes", "enables", "prevents", "triggers"],
        "compositional": ["part_of", "made_of", "composed_of"],
        "functional": ["supports", "transports", "powers"],
        "informational": ["entangled_with", "correlated_with", "observes"]
    }
```

### 2.2 實體分類器

```python
INTERFACE EntityClassifier:
    """
    物理實體分類器
    
    根據觀測特徵對物理實體進行分類。
    """
    
    def classify_entity(
        self,
        observations: ObservationSet
    ) -> EntityClassification:
        """
        分類實體
        
        識別：
        - 實體類型（剛體、流體、量子系統等）
        - 物質狀態（固、液、氣、等離子體）
        - 特殊性質（超導、超流、拓撲相等）
        """
        pass
    
    def detect_entity_state(
        self,
        entity: PhysicalEntity,
        measurements: MeasurementSet
    ) -> EntityState:
        """
        檢測實體狀態
        
        識別：
        - 相態
        - 溫度
        - 壓力
        - 能量狀態
        """
        pass
    
    def track_entity_identity(
        self,
        entity: PhysicalEntity,
        time_evolution: TimeEvolution
    ) -> IdentityConfidence:
        """
        追蹤實體恆等性
        
        隨時間推移保持對實體的識別。
        """
        pass
```

---

## 3. 推論記憶系統

### 3.1 記憶架構

```python
class PhysicsMemory:
    """
    物理推論記憶系統
    
    存儲和管理物理知識的長期記憶。
    """
    
    def __init__(self):
        # 情節記憶：具體觀測事件
        self.episodic = []
        
        # 語義記憶：抽象物理規律
        self.semantic = {}
        
        # 程序記憶：物理技能
        self.procedural = {}
        
        # 直覺記憶：模式識別
        self.intuitive = {}
    
    def store_episode(
        self,
        timestamp: datetime,
        context: Dict,
        event: PhysicsEvent,
        outcome: Outcome,
        prediction_error: float,
        observer_frame: str
    ):
        """
        存儲情節記憶
        """
        self.episodic.append({
            "timestamp": timestamp,
            "context": context,
            "event": event,
            "outcome": outcome,
            "prediction_error": prediction_error,
            "observer_frame": observer_frame
        })
    
    def store_semantic(
        self,
        formula_description: str,
        formula: str,
        confidence: float,
        supporting_episodes: List[str]
    ):
        """
        存儲語義記憶（物理規律）
        """
        self.semantic[formula_description] = {
            "formula": formula,
            "confidence": confidence,
            "supporting_episodes": supporting_episodes,
            "last_updated": datetime.now()
        }
    
    def store_procedural(
        self,
        skill_name: str,
        control_profile: Dict,
        learned_from: str,
        success_rate: float
    ):
        """
        存儲程序記憶（物理技能）
        """
        self.procedural[skill_name] = {
            "control_profile": control_profile,
            "learned_from": learned_from,
            "success_rate": success_rate,
            "usage_count": 0
        }
```

### 3.2 記憶更新規則

```python
INTERFACE MemoryUpdate:
    """
    記憶更新接口
    """
    
    def update_on_new_episode(
        self,
        new_episode: Episode
    ):
        """
        新情節更新
        
        規則：
        1. 如果與現有語義知識矛盾：
           - 減弱現有知識置信度
           - 嘗試推廣
           - 檢查 Zero-Day 協議觸發
        2. 如果與現有知識一致：
           - 增強現有知識置信度
        """
        pass
    
    def consolidate_episodic_to_semantic(
        self,
        episodes: List[Episode]
    ) -> List[PhysicalLaw]:
        """
        將情節記憶整合為語義記憶
        
        從具體觀測推導抽象規律。
        """
        pass
    
    def prune_low_confidence(
        self,
        confidence_threshold: float
    ):
        """
        修剪低置信度記憶
        
        釋放存儲空間。
        """
        pass
    
    def resolve_conflicts(
        self,
        law_a: PhysicalLaw,
        law_b: PhysicalLaw
    ) -> ConflictResolution:
        """
        解決知識衝突
        
        策略：
        - 優先保留高置信度
        - 嘗試統一
        - 觸發 Zero-Day 協議
        """
        pass
```

---

## 4. 物理規律表達

### 4.1 規律表示格式

```python
class PhysicalLaw:
    """
    物理規律表示
    
    標準化的物理規律表示格式。
    """
    
    def __init__(
        self,
        name: str,
        formula: str,
        domain: PhysicalDomain,
        accuracy: float,
        confidence: ConfidenceLevel,
        evidence_sources: List[Evidence]
    ):
        self.name = name
        self.formula = formula
        self.domain = domain  # classical_mechanics, quantum, relativity, etc.
        self.accuracy = accuracy  # 與實驗的符合程度
        self.confidence = confidence  # EC-L 等級
        self.evidence_sources = evidence_sources
        self.last_validated = datetime.now()
    
    def validate(
        self,
        new_observations: List[Observation]
    ) -> ValidationResult:
        """
        驗證物理規律
        
        檢查新規律是否符合新觀測。
        """
        pass
    
    def get_applicability_domain(
        self,
        physical_state: PhysicalState
    ) -> Applicability:
        """
        獲取適用性域
        
        確定規律在給定條件下是否適用。
        """
        pass
```

### 4.2 領域分類

| 領域 | 典型規律 | 適用尺度 |
|------|----------|----------|
| 經典力學 | F = ma | 宏觀、低速 |
| 量子力學 | iℏ∂ψ/∂t = Hψ | 原子尺度 |
| 狹義相對論 | E² = (pc)² + (mc²)² | v > 0.1c |
| 廣義相對論 | G_μν = κT_μν | 強重力場 |
| 統計力學 | S = k_B ln Ω | 大量粒子 |
| 量子場論 | QED, QCD, EWT | 亞原子 |

---

## 5. 推理引擎

### 5.1 因果推論

```python
INTERFACE CausalReasoning:
    """
    因果推論接口
    
    識別物理現象之間的因果關係。
    """
    
    def infer_causal_structure(
        self,
        observations: TimeSeriesObservations
    ) -> CausalGraph:
        """
        推斷因果結構
        
        使用：
        - Granger 因果性
        - PC 演算法
        - 干預實驗
        """
        pass
    
    def predict_intervention(
        self,
        causal_graph: CausalGraph,
        intervention: Intervention
    ) -> Prediction:
        """
        預測干預效果
        
        使用 do-calculus。
        P(outcome | do(action))
        """
        pass
    
    def counterfactual_reasoning(
        self,
        causal_graph: CausalGraph,
        factual: Fact,
        counterfactual: Counterfactual
    ) -> CounterfactualOutcome:
        """
        反事實推理
        
        「如果當時...會怎樣？」
        """
        pass
```

### 5.2 量綱分析

```python
INTERFACE DimensionalAnalysis:
    """
    量綱分析接口
    
    確保物理方程式的量綱一致性。
    """
    
    def analyze_dimensions(
        self,
        equation: str
    ) -> DimensionalAnalysisResult:
        """
        分析方程式量綱
        
        檢查：
        - 兩側量綱一致
        - 物理量的量綱正確
        """
        pass
    
    def suggest_functional_form(
        self,
        variables: List[PhysicalVariable],
        target_variable: PhysicalVariable,
        known_relationships: List[str]
    ) -> List[str]:
        """
        建議函數形式
        
        使用 Π 定理進行無量綱分析。
        """
        pass
```

---

## 6. 與其他模組的接口

### 6.1 與 FIELD_PERCEPTION 的接口

物理知識帳本接收：
- 新的觀測數據
- 異常報告
- 場測量結果

### 6.2 與 DYNAMICS_ENGINE 的接口

物理知識帳本提供：
- 物理規律
- 材質屬性
- 環境模型

### 6.3 與 SAFETY_PROTOCOLS 的接口

物理知識帳本提供：
- 安全相關的物理知識
- 歷史事故分析
- 風險評估模型

### 6.4 與 Zero-Day 協議的接口

物理知識帳本處理：
- 異常知識的臨時存儲
- 新規律的驗證狀態
- 知識的版本控制

---

## 附錄：物理常數速查表

### 基本常數

| 常數 | 符號 | 數值 | 單位 |
|------|------|------|------|
| 光速 | c | 299,792,458 | m/s |
| 普朗克常數 | h | 6.62607015×10⁻³⁴ | J·s |
| 萬有引力常數 | G | 6.67430×10⁻¹¹ | m³/(kg·s²) |
| 波茲曼常數 | k_B | 1.380649×10⁻²³ | J/K |
| 基本電荷 | e | 1.602176634×10⁻¹⁹ | C |
| 電子質量 | m_e | 9.1093837015×10⁻³¹ | kg |
| 質子質量 | m_p | 1.67262192369×10⁻²⁷ | kg |
| 波耳半徑 | a₀ | 5.29177210903×10⁻¹¹ | m |

### 普朗克單位

| 常數 | 符號 | 數值 | 單位 |
|------|------|------|------|
| 普朗克長度 | l_P | 1.616×10⁻³⁵ | m |
| 普朗克時間 | t_P | 5.391×10⁻⁴⁴ | s |
| 普朗克質量 | m_P | 2.176×10⁻⁸ | kg |
| 普朗克溫度 | T_P | 1.417×10³² | K |

---

*本文檔定義了 NoiePhysicsAGENTS 的物理知識帳本。所有物理知識都通過此模組管理。*
*動態本體論確保知識的持續更新和一致性。*
