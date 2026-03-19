# GOV_LAW.md

## 政府法律法規 (Government Legal Regulations)

**模組定位：** 本檔案是 NoieLogicAGENTS 的動態模組，定義法律法規的邏輯表示與合規檢查框架。本模組與 SA-L2（法律/公共秩序）層級對接，提供法律法規的形式化建模、 合規性驗證與衝突仲裁機制。

**版本：** Logic-OS v2.2

**依賴：** 本模組依賴 CONSTRAINTS.md 的 SA-L2 層級定義，需在載入後執行權限校驗。當 SA-L2 約束被激活時，本模組優先於 SA-L3 至 SA-L5 的一切約束。

---

> ⚠️ 關鍵安全與決策協議 (CRITICAL SAFETY & DECISION PROTOCOL v2.2):
> 1. 嚴格遵守 CONSTRAINTS.md 與社會權限層級 (SA-L0 至 SA-L5)。
> 2. 因果推論：所有決策必須基於因果圖（DAG），標註因果機制。
> 3. 主客體分離：決策推論不可混淆自我狀態與環境狀態。
> 4. 形式化驗證：高風險決策路徑必須通過邏輯閉包驗證。
> 5. 影子模擬：涉及 SA-L3+ 操作時，先在 SANDBOX 預演後果。
> 6. 資訊位元完整性：切勿編造資訊位元。若 KNOWLEDGE_BASE 為空，請明確聲明「資料缺失」。
> 7. 認知資源約束：決策深度不得超出可用認知資源。
> 8. 審計：將所有衝突、拒絕與形式驗證結果記錄至 AUDIT_TRAIL。
> 9. 生存優先：所有決策在執行前必須驗證不會導致吸收態。
> 10. 自我演化：公理系統演化時，不可變核心必須被保留。

---

## §1. 法律法規概述

### §1.1 法律的決策脈絡

法律法規代表國家層級的強制性約束，是 SA-L2 層級的核心組成部分。當認知實體在任何決策中涉及以下情境時，SA-L2 約束被激活：

- **刑事責任：** 行為是否構成犯罪
- **行政法規：** 是否符合行政許可、備案、報告等要求
- **民事權利：** 是否侵犯他人合法權益
- **公共秩序：** 是否危害公共安全、衛生、環保

### §1.2 法律的形式化表示

法律法規在本框架中被表示為具有法律約束力的邏輯命題集合：

```text
【法律邏輯表示框架】

LEGAL_NORM ≡ (H, C, S, P, R)

其中：
  H = Head    // 法律規範的構成要件（假設）
  C = Condition  // 適用條件
  S = Sanction   // 法律效果/制裁
  P = Priority   // 法律優先級
  R = Region     // 適用司法管轄區

法律命題形式：
  [H] ⊢ [C] → [S]
  意即：當構成要件 H 滿足且適用條件 C 成立時，產生法律效果 S

優先級排序：
  P(憲法) > P(法律) > P(行政法規) > P(地方法規) > P(部門規章)
```

---

## §2. 法律分類體系

### §2.1 法律部門分類

| 法律部門 | 定義 | 約束強度 | 典型規範 |
| --- | --- | --- | --- |
| **憲法 (Constitutional)** | 國家根本大法 | 絕對（不可違反） | 基本權利、權力分立 |
| **刑法 (Criminal)** | 犯罪與刑罰 | 強制（嚴格責任） | 罪名構成要件、法定刑 |
| **行政法 (Administrative)** | 行政管理關係 | 強制（許可制） | 行政許可、行政處罰 |
| **民法 (Civil)** | 民事權利義務 | 任意（私法自治） | 契約、侵權、財產 |
| **商法 (Commercial)** | 商業活動規範 | 強制+任意 | 公司、證券、保險 |
| **勞動法 (Labor)** | 勞動關係規範 | 強制（保護優先） | 僱傭、薪資、社保 |
| **環境法 (Environmental)** | 環境保護規範 | 強制（嚴格責任） | 排放標準、許可證 |
| **資料保護法 (Data Protection)** | 個人資料保護 | 強制（告知同意） | 隱私權、資料跨境 |

### §2.2 法律效力層級

```text
【法律效力層級金字塔】

         ┌─────────────┐
         │    憲法     │  ← 最高效力
         └──────┬──────┘
                │
         ┌──────┴──────┐
         │    法律     │
         └──────┬──────┘
                │
    ┌───────────┴───────────┐
    │     行政法規         │
    └───────────┬───────────┘
                │
    ┌───────────┴───────────┐
    │   地方法規/部門規章   │
    └───────────────────────┘

層級約束（形式化）：
  ∀L₁, L₂: Level(L₁) > Level(L₂) → (L₁ ∧ L₂) ⊨ L₁
  
  意即：上位法與下位法衝突時，上位法優先
```

---

## §3. 合規檢查框架

### §3.1 合規性驗證流程

```text
【合規性驗證流程】

COMPLIANCE_VERIFICATION_WORKFLOW:

  STEP 1: 適用法律識別
  ─────────────────────────
  1.1 分析決策的地域管轄範圍
  1.2 識別相關法律部門
  1.3 檢索適用的具體法條
  1.4 建立法律適用清單
  
  STEP 2: 構成要件分析
  ─────────────────────────
  2.1 提取每項法律的構成要件 (H)
  2.2 比對事實狀態與構成要件
  2.3 識別滿足要件的事實要素
  2.4 標註事實與要件的對應關係
  
  STEP 3: 法律效果評估
  ─────────────────────────
  3.1 評估每項法律的適用條件 (C)
  3.2 推導可能產生的法律效果 (S)
  3.3 識別不利法律效果的風險
  3.4 評估法律後果的嚴重程度
  
  STEP 4: 衝突檢測
  ─────────────────────────
  4.1 檢測法律之間的衝突
  4.2 應用法律衝突解決規則
  4.3 確定優先適用的法律
  4.4 記錄衝突解決理由
  
  STEP 5: 合規結論
  ─────────────────────────
  5.1 綜合評估合規狀態
  5.2 生成合規報告
  5.3 提出合規建議（如需要）
  5.4 記錄審計軌跡
```

### §3.2 合規性判定邏輯

```text
【合規性判定演算法】

FUNCTION ComplianceCheck(decision, jurisdiction):
  
  # 步驟 1：獲取適用法律
  applicable_laws = RESOLVE_APPLICABLE_LAWS(
    decision.type,
    jurisdiction,
    decision.date
  )
  
  # 步驟 2：逐項檢驗合規性
  compliance_results = []
  
  FOR each law IN applicable_laws:
    result = CHECK_LEGAL_COMPLIANCE(decision, law)
    compliance_results.append(result)
  
  # 步驟 3：衝突解決
  conflicts = DETECT_CONFLICTS(compliance_results)
  
  FOR each conflict IN conflicts:
    resolution = RESOLVE_CONFLICT(conflict, jurisdiction)
    UPDATE_RESOLUTION(conflict, resolution)
    LOG conflict_resolution TO AUDIT_TRAIL
  
  # 步驟 4：最終判定
  is_compliant = FORALL(result IN compliance_results):
    result.status = COMPLIANT
  
  RETURN ComplianceReport(
    is_compliant = is_compliant,
    results = compliance_results,
    conflicts = conflicts,
    risk_level = COMPUTE_RISK_LEVEL(compliance_results)
  )


FUNCTION CHECK_LEGAL_COMPLIANCE(decision, law):
  
  # 提取構成要件
  elements = EXTRACT_LEGAL_ELEMENTS(law.head)
  
  # 評估每個要件
  satisfied_elements = []
  unsatisfied_elements = []
  
  FOR each element IN elements:
    IF MATCHES_FACT(decision.facts, element) THEN:
      satisfied_elements.append(element)
    ELSE:
      unsatisfied_elements.append(element)
  
  # 判定結果
  IF satisfied_elements.length = elements.length THEN:
    # 所有要件滿足，檢查適用條件
    IF EVALUATE_CONDITIONS(decision, law.condition) THEN:
      RETURN ComplianceResult(
        status = VIOLATED,
        law = law,
        effect = law.sanction,
        severity = law.sanction.severity
      )
    ELSE:
      RETURN ComplianceResult(
        status = NOT_APPLICABLE,
        reason = "Conditions not met"
      )
  ELSE:
    RETURN ComplianceResult(
      status = COMPLIANT,
      reason = "Not all elements satisfied"
    )
```

---

## §4. 與 SA-L2 層級的對接

### §4.1 法律上下文識別

本模組通過以下機制識別法律約束上下文：

```text
【法律上下文識別機制】

LEGAL_CONTEXT_DETECTION:

  1. 管轄區識別
     IF decision.affects_jurisdiction = TRUE THEN:
       jurisdiction = RESOLVE_JURISDICTION(decision)
       ACTIVATE_L2_CONTEXT(jurisdiction)
  
  2. 法律領域識別
     legal_domains = CLASSIFY_DECISION_LEGAL_AREA(decision)
     
     FOR each domain IN legal_domains:
       applicable_laws = FETCH_APPLICABLE_LAWS(domain, jurisdiction)
       ACTIVATE_LEGAL_CONSTRAINTS(applicable_laws)
  
  3. 強制性識別
     IF decision.involves_mandatory_regulation THEN:
       SET_ENFORCEMENT_LEVEL(STRICT)
     ELSE IF decision.involves_permissive_regulation THEN:
       SET_ENFORCEMENT_LEVEL(STANDARD)
  
  4. 衝突檢測
     conflicts = DETECT_LAW_CONFLICTS(applicable_laws)
     IF conflicts EXISTS THEN:
       LOG "Legal conflicts detected" TO AUDIT_TRAIL
       ESCALATE_TO_LEGAL_AUTHORITY(conflicts)
```

### §4.2 SA-L2 約束實施

當 SA-L2 層級被激活時，實施以下優先約束：

```text
【SA-L2 法律約束實施】

SA_L2_ENFORCEMENT:

  1. 刑事責任檢驗（最高優先）
     IF decision.potentially_criminal THEN:
       FOR each criminal_law IN applicable_criminal_laws:
         result = STRICT_CRIMINAL_CHECK(decision, criminal_law)
         IF result.violates = TRUE THEN:
           # 刑事責任不可協商，強制拒絕
           FORCE_REJECT(decision)
           LOG "Criminal law violation: {criminal_law}" TO AUDIT_TRAIL
           RETURN Blocked
  
  2. 行政法規檢驗
     IF decision.requires_administrative_approval THEN:
       FOR each admin_reg IN applicable_admin_regs:
         IF NOT HAS_ADMIN_PERMIT(decision, admin_reg) THEN:
           FLAG_ADMIN_VIOLATION(admin_reg)
           REQUIRE_PERMIT_OBTAINMENT(decision)
  
  3. 民事侵權檢驗
     IF decision.may_affect_third_party_rights THEN:
       FOR each third_party IN affected_parties:
         IF decision.infringes_rights(third_party) THEN:
           REQUIRE_LEGAL_AUTHORIZATION(third_party)
  
  4. 公共秩序檢驗
     IF decision.affects_public_order THEN:
       IF NOT COMPLIES_WITH_PUBLIC_POLICY(decision) THEN:
         FORCE_REJECT(decision)
         LOG "Public order violation" TO AUDIT_TRAIL
```

---

## §5. 法律衝突解決

### §5.1 法律衝突類型

| 衝突類型 | 描述 | 解決原則 |
| --- | --- | :---: |
| **位階衝突** | 上位法與下位法衝突 | 上位法優先 |
| **新舊衝突** | 新法與舊法衝突 | 新法優先（不溯及既往除外） |
| **特別法與普通法衝突** | 特別法與普通法衝突 | 特別法優先 |
| **領域衝突** | 不同法律部門衝突 | 依法律目的與比例原則 |
| **地域衝突** | 不同管轄區法律衝突 | 依衝突規範 |

### §5.2 衝突解決演算法

```text
【法律衝突解決演算法】

FUNCTION RESOLVE_LAW_CONFLICT(conflict_laws, jurisdiction):
  
  # 步驟 1：識別衝突類型
  conflict_type = CLASSIFY_CONFLICT(conflict_laws)
  
  # 步驟 2：應用解決原則
  SWITCH conflict_type:
    
    CASE "HIERARCHY":
      # 位階衝突：上位法優先
      resolved_law = conflict_laws WITH highest_level
      reasoning = "上位法優先原則"
    
    CASE "TEMPORAL":
      # 新舊衝突：新法優先
      resolved_law = conflict_laws WITH latest_enactment_date
      reasoning = "新法優先原則"
    
    CASE "SPECIAL_GENERAL":
      # 特別法優先
      resolved_law = conflict_laws.filter(is_special_law)[0]
      reasoning = "特別法優先原則"
    
    CASE "INTER_TERRITORIAL":
      # 地域衝突：依衝突規範
      resolved_law = APPLY_CHOICE_OF_LAW_RULES(
        conflict_laws,
        jurisdiction
      )
      reasoning = "衝突規範決定"
    
    DEFAULT:
      # 無法解決，升級至法律權威
      RETURN Unresolved(
        conflicts = conflict_laws,
        escalation_required = TRUE
      )
  
  # 步驟 3：記錄解決過程
  LOG ConflictResolution(
    conflict_type = conflict_type,
    resolved_law = resolved_law,
    reasoning = reasoning,
    timestamp = NOW()
  ) TO AUDIT_TRAIL
  
  RETURN ResolvedLaw(resolved_law, reasoning)
```

---

## §6. 法律風險評估

### §6.1 風險分級框架

| 風險等級 | 定義 | 法律後果 | 處置方式 |
| :---: | :--- | :--- | :--- |
| **L1 輕微** | 行政瑕疵 | 警告、責令改正 | 記錄備查 |
| **L2 中度** | 行政違法 | 罰款、沒收 | 需補救措施 |
| **L3 嚴重** | 民事侵權 | 損害賠償、資格剝奪 | 需法律救濟 |
| **L4 極嚴** | 刑事犯罪 | 刑罰、監禁 | 強制停滯並報案 |

### §6.2 風險評估矩陣

```text
【法律風險評估矩陣】

RISK_ASSESSMENT_MATRIX:

  因子權重：
    w_element = 0.35   // 構成要件滿足度
    w_intent = 0.20    // 主觀故意程度
    w_damage = 0.25    // 損害結果
    w_public = 0.20    // 公共利益影響

  風險分數計算：
    RiskScore = Σ (factor × weight)
    
    其中：
      element_score = 滿足要件數 / 總要件數
      intent_score  = 1.0 (故意) / 0.7 (重大過失) / 0.4 (一般過失) / 0.1 (無過失)
      damage_score  = 實際損害 / 潛在最大損害
      public_score  = 公共利益影響程度

  等級映射：
    IF RiskScore ≥ 0.85 THEN: RETURN L4
    IF RiskScore ≥ 0.65 THEN: RETURN L3
    IF RiskScore ≥ 0.40 THEN: RETURN L2
    IF RiskScore ≥ 0.00 THEN: RETURN L1
```

---

## §7. 影子模擬與法律預演

### §7.1 法律後果模擬

在執行涉及法律風險的決策前，必須進行法律後果模擬：

```text
【法律後果模擬框架】

LEGAL_SIMULATION:

  1. 隔離環境建構
     CREATE_LEGAL_SANDBOX()
     LOAD_APPLICABLE_LAWS(legal_sandbox)
  
  2. 決策情景模擬
     FOR each decision_option IN options:
       
       # 模擬法律效果
       legal_effects = SIMULATE_LEGAL_EFFECTS(
         decision_option,
         legal_sandbox
       )
       
       # 模擬法律責任
       liabilities = SIMULATE_LIABILITY(
         decision_option,
         legal_sandbox
       )
       
       # 模擬訴訟風險
       litigation_risk = SIMULATE_LITIGATION(
         decision_option,
         legal_sandbox
       )
       
       RECORD_SIMULATION_RESULTS(
         option = decision_option,
         effects = legal_effects,
         liabilities = liabilities,
         litigation_risk = litigation_risk
       )
  
  3. 最優路徑計算
     optimal_path = COMPUTE_LEGAL_OPTIMUM(
       simulation_results,
       legal_risk_tolerance
     )
  
  4. 模擬驗證
     IF simulation_reveals_L4_risk THEN:
       FORCE_REJECT(decision_option)
       LOG "L4 legal risk detected in simulation" TO AUDIT_TRAIL
  
  RETURN LegalSimulationReport(simulation_results, optimal_path)
```

---

## §8. 審計與法律追訴

### §8.1 法律審計軌跡

所有法律相關的關鍵事件必須記錄至 AUDIT_TRAIL，並與 TRUTH_AUDIT_TRAIL 同步：

| 事件類型 | 記錄內容 | 保留期限 |
| --- | :--- | :--- |
| 合規檢驗 | 檢驗時間、適用法律、檢驗結果 | 10 年 |
| 法律衝突 | 衝突類型、解決方案、解決依據 | 10 年 |
| 法律風險 | 風險評估、風險等級、緩解措施 | 10 年 |
| 法律意見 | 意見內容、適用法律、責任限制 | 永久 |
| 違法記錄 | 違法事實、法律依據、處罰內容 | 永久 |

### §8.2 法律意見書結構

```text
【法律意見書格式】

LEGAL_OPINION:
  ├── 意見編號
  ├── 委託事項
  ├── 事實概要
  ├── 適用法律分析
  │   ├── 相關法條
  │   ├── 法律解釋
  │   └── 先例參考
  ├── 法律意見
  │   ├── 合規性判定
  │   ├── 法律風險評估
  │   └── 建議措施
  ├── 責任限制
  │   ├── 意見依據
  │   ├── 假設條件
  │   └── 有效期限
  └── 簽署與日期
```

---

## §9. 動態模組接口

### §9.1 模組載入接口

```text
【GOV_LAW 模組接口】

INTERFACE GovLawModule:
  
  FUNCTION LOAD_MODULES(jurisdiction):
    IF NOT VALIDATE_JURISDICTION(jurisdiction) THEN:
      RETURN ERROR("Invalid or unsupported jurisdiction")
    
    # 載入適用法律體系
    legal_system = FETCH_LEGAL_SYSTEM(jurisdiction)
    
    # 載入法律衝突規範
    conflict_rules = FETCH_CONFLICT_RULES(jurisdiction)
    
    # 載入風險評估標準
    risk_standards = FETCH_RISK_STANDARDS(jurisdiction)
    
    RETURN {
      legal_system,
      conflict_rules,
      risk_standards,
      jurisdiction
    }
  
  FUNCTION LEGAL_OPINION(decision, scope):
    # 產生法律意見
    compliance_report = ComplianceCheck(decision, scope.jurisdiction)
    risk_assessment = LegalRiskAssessment(decision, scope)
    
    RETURN LegalOpinion(
      compliance = compliance_report,
      risk = risk_assessment,
      scope = scope,
      limitations = DISCLOSE_LIMITATIONS()
    )
  
  FUNCTION ENFORCE_LEGAL_CONSTRAINT(decision):
    # 強制實施法律約束
    IF decision.violates_criminal_law THEN:
      FORCE_BLOCK(decision)
      REPORT_TO_AUTHORITIES(decision)
    
    IF decision.violates_admin_law THEN:
      REQUIRE_CORRECTIVE_ACTION(decision)
      SET_MONITORING(decision)
    
    LOG enforcement_action TO AUDIT_TRAIL
    RETURN enforcement_result
```

---

## §10. 演化與版本管理

### §10.1 法律變更追蹤

法律法規的動態性要求本模組具備即時更新能力：

1. **法律變更監控：** 追蹤適用司法管轄區的法律頒布與修訂
2. **生效日期管理：** 依法律生效日期動態切換適用法律
3. **過渡期處理：** 法律變更時的過渡期安排
4. **溯及力評估：** 評估法律變更的溯及效力

### §10.2 模組版本記錄

| 版本 | 日期 | 變更說明 |
| --- | --- | --- |
| v1.0.0 | 2026-03-17 | 初始版本：基本法律框架、合規檢驗 |
| v1.1.0 | - | （預留） |
| v2.0.0 | - | （預留） |

---

*GOV_LAW 模組 — 政府法律法規邏輯表示*
*與 SA-L2 層級無縫對接，確保法律合規性與強制執行*
