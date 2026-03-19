# NoieLogicAGENTS — 一致性驗證引擎

**版本：** Logic-OS v2.2  
**模組代號：** CONSISTENCY_ENGINE  
**職責：** 矛盾偵測、一致性維護、衝突解決  
**父模組：** FORMAL_VERIFIER

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

## §0. 一致性框架

### §0.1 核心定義

```text
【一致性定義】

一個命題集合 S 是一致的，當且僅當：

  ¬∃φ: (φ ∈ S ∧ ¬φ ∈ S)

即：不存在命題 φ 使得 φ 和 ¬φ 都在 S 中為真。

【矛盾類型】

1. 命題矛盾 (Propositional Contradiction)
   - 直接形式：p ∧ ¬p
   - 例子：「今天下雨」且「今天不下雨」

2. 包含矛盾 (Set-theoretic Contradiction)
   - 形式：A ⊂ B 且 B ⊂ A
   - 例子：「所有鳥都會飛」且「所有會飛的都是鳥」

3. 量化矛盾 (Quantifier Contradiction)
   - 形式：∃x:P(x) ∧ ∀x:¬P(x)
   - 例子：「有人作弊」且「所有人都沒作弊」

4. 語境矛盾 (Contextual Contradiction)
   - 形式：在上下文 C₁ 中 φ 成立，在 C₂ 中 ¬φ 成立
   - 例子：不同決策假設下的矛盾結論

5. 層級矛盾 (Hierarchy Contradiction)
   - 形式：SA-Ln 層級的決策與 SA-Lm 層級的約束衝突
   - 例子：個人決策違反組織政策
```

### §0.2 一致性層級

```text
【一致性層級】

| 層級 | 名稱 | 定義 | 應用場景 |
|------|------|------|----------|
| CL-0 | 完美一致 | 無任何矛盾 | SA-L0 生存決策 |
| CL-1 | 局部一致 | 可分割子集內一致 | SA-L1 憲法級 |
| CL-2 | 上下文一致 | 單一上下文內一致 | SA-L2 法律級 |
| CL-3 | 寬容一致 | 允許暫時不一致 | SA-L3 組織級 |
| CL-4 | 動態一致 | 演化過程中趨向一致 | SA-L4 家庭級 |
| CL-5 | 近似一致 | 允許近義矛盾 | SA-L5 個人級 |
```

---

## §1. 矛盾偵測

### §1.1 矛盾偵測引擎

```python
class ContradictionDetector:
    """
    矛盾偵測引擎
    
    全面檢測各類型矛盾
    """
    
    def __init__(self):
        self.propositional_checker = PropositionalContradictionChecker()
        self.set_checker = SetContradictionChecker()
        self.quantifier_checker = QuantifierContradictionChecker()
        self.contextual_checker = ContextualContradictionChecker()
        self.hierarchy_checker = HierarchyContradictionChecker()
    
    def detect_contradictions(self, proposition_set: Set[Proposition], 
                            context: DecisionContext = None) -> ContradictionResult:
        """
        偵測命題集合中的所有矛盾
        
        返回：
            ContradictionResult，包含：
            - has_contradiction: 是否存在矛盾
            - contradictions: 矛盾列表
            - severity: 嚴重性評估
            - resolution_suggestions: 解決建議
        """
        
        all_contradictions = []
        
        # 1. 命題矛盾偵測
        propositional_contradictions = \
            self.propositional_checker.check(proposition_set)
        all_contradictions.extend(propositional_contradictions)
        
        # 2. 集合矛盾偵測
        set_contradictions = self.set_checker.check(proposition_set)
        all_contradictions.extend(set_contradictions)
        
        # 3. 量化矛盾偵測
        quantifier_contradictions = self.quantifier_checker.check(proposition_set)
        all_contradictions.extend(quantifier_contradictions)
        
        # 4. 語境矛盾偵測（如果提供上下文）
        if context:
            contextual_contradictions = self.contextual_checker.check(
                proposition_set, context
            )
            all_contradictions.extend(contextual_contradictions)
        
        # 5. 層級矛盾偵測
        hierarchy_contradictions = self.hierarchy_checker.check(proposition_set)
        all_contradictions.extend(hierarchy_contradictions)
        
        # 計算嚴重性
        severity = self._calculate_severity(all_contradictions)
        
        # 生成解決建議
        suggestions = self._generate_suggestions(all_contradictions)
        
        return ContradictionResult(
            has_contradiction=len(all_contradictions) > 0,
            contradictions=all_contradictions,
            count=len(all_contradictions),
            severity=severity,
            resolution_suggestions=suggestions
        )
    
    def _calculate_severity(self, contradictions: List[Contradiction]) -> str:
        """
        計算矛盾的整體嚴重性
        """
        
        if not contradictions:
            return 'NONE'
        
        severity_levels = {'CRITICAL': 4, 'HIGH': 3, 'MEDIUM': 2, 'LOW': 1}
        
        max_severity = max(
            severity_levels.get(c.severity, 0) 
            for c in contradictions
        )
        
        for level, value in severity_levels.items():
            if max_severity == value:
                return level
        
        return 'UNKNOWN'
    
    def _generate_suggestions(self, contradictions: List[Contradiction]) -> List[str]:
        """
        根據矛盾類型生成解決建議
        """
        
        suggestions = []
        
        for contradiction in contradictions:
            if contradiction.type == 'propositional':
                suggestions.append(
                    f"命題矛盾：檢查 {contradiction.proposition_a} 和 {contradiction.proposition_b}，"
                    "可能需要修正其中一個命題"
                )
            elif contradiction.type == 'set_theoretic':
                suggestions.append(
                    f"包含矛盾：審查集合 {contradiction.set_a} 和 {contradiction.set_b} 的邊界定義"
                )
            elif contradiction.type == 'quantifier':
                suggestions.append(
                    f"量化矛盾：檢查 {contradiction.existence_claim} 與 {contradiction.universal_claim} 的範圍"
                )
            elif contradiction.type == 'contextual':
                suggestions.append(
                    f"語境矛盾：區分 {contradiction.context_a} 和 {contradiction.context_b} 的適用範圍"
                )
            elif contradiction.type == 'hierarchy':
                suggestions.append(
                    f"層級矛盾：SA-L{contradiction.sa_level_a} 與 SA-L{contradiction.sa_level_b} 衝突，"
                    "需要升級至更高層級仲裁"
                )
        
        return suggestions
```

### §1.2 命題矛盾偵測

```python
class PropositionalContradictionChecker:
    """
    命題矛盾偵測器
    
    偵測命題邏輯層面的矛盾
    """
    
    def check(self, propositions: Set[Proposition]) -> List[Contradiction]:
        """
        偵測命題矛盾
        
        策略：
        1. 直接矛盾：p 和 ¬p
        2. 蘊含矛盾：p → q 和 p → ¬q（當 p 為真時）
        3. 等價矛盾：p ↔ q 和 p ↔ ¬q
        """
        
        contradictions = []
        proposition_list = list(propositions)
        
        # 策略 1：直接矛盾檢測
        for i, prop_a in enumerate(proposition_list):
            for prop_b in proposition_list[i+1:]:
                # 檢查是否是直接矛盾
                if self._is_direct_contradiction(prop_a, prop_b):
                    contradictions.append(Contradiction(
                        type='propositional',
                        subtype='direct',
                        proposition_a=str(prop_a),
                        proposition_b=str(prop_b),
                        severity='CRITICAL',
                        description=f"直接矛盾：{prop_a} 和 {prop_b}"
                    ))
                
                # 檢查是否是蘊含矛盾
                implied_contradiction = self._check_implicative_contradiction(
                    prop_a, prop_b, proposition_list
                )
                if implied_contradiction:
                    contradictions.append(implied_contradiction)
        
        return contradictions
    
    def _is_direct_contradiction(self, prop_a: Proposition, prop_b: Proposition) -> bool:
        """
        檢查是否是直接矛盾
        
        形式：p 和 ¬p
        """
        
        # 檢查 prop_b 是否是 prop_a 的否定
        if prop_b == Proposition.negate(prop_a):
            return True
        
        # 檢查 prop_a 是否是 prop_b 的否定
        if prop_a == Proposition.negate(prop_b):
            return True
        
        # 檢查簡化形式
        if prop_a.simplified_form == Proposition.negate(prop_b).simplified_form:
            return True
        
        return False
    
    def _check_implicative_contradiction(self, prop_a: Proposition, 
                                         prop_b: Proposition,
                                         all_propositions: List[Proposition]) -> Contradiction:
        """
        檢查蘊含矛盾
        
        形式：p → q 和 p → ¬q（當 p 為真時）
        """
        
        # 找到 p → q 形式
        if prop_a.is_conditional and prop_b.is_conditional:
            # 檢查前件是否相同
            if prop_a.antecedent == prop_b.antecedent:
                # 檢查後件是否矛盾
                if prop_b.consequent == Proposition.negate(prop_a.consequent):
                    return Contradiction(
                        type='propositional',
                        subtype='implicative',
                        proposition_a=str(prop_a),
                        proposition_b=str(prop_b),
                        severity='HIGH',
                        description=f"蘊含矛盾：當 {prop_a.antecedent} 為真時無法同時導出 "
                                   f"{prop_a.consequent} 和 {prop_b.consequent}"
                    )
        
        return None
```

### §1.3 集合矛盾偵測

```python
class SetContradictionChecker:
    """
    集合矛盾偵測器
    
    偵測集合論層面的矛盾
    """
    
    def check(self, propositions: Set[Proposition]) -> List[Contradiction]:
        """
        偵測集合矛盾
        
        形式：
        1. A ⊂ B 且 B ⊂ A（循環包含）
        2. A = B 且 A ≠ B
        3. x ∈ A 且 x ∉ A
        """
        
        contradictions = []
        
        # 提取命題中的集合關係
        set_relations = self._extract_set_relations(propositions)
        
        # 檢查循環包含
        for (set_a, set_b), relation_type in set_relations.items():
            if relation_type == 'subset':
                # 檢查反向包含
                reverse_key = (set_b, set_a)
                if reverse_key in set_relations and \
                   set_relations[reverse_key] == 'subset':
                    contradictions.append(Contradiction(
                        type='set_theoretic',
                        subtype='circular_subset',
                        set_a=set_a,
                        set_b=set_b,
                        severity='CRITICAL',
                        description=f"循環包含：{set_a} ⊂ {set_b} 且 {set_b} ⊂ {set_a}"
                    ))
        
        return contradictions
    
    def _extract_set_relations(self, propositions: Set[Proposition]) -> dict:
        """
        從命題中提取集合關係
        """
        
        relations = {}
        
        for prop in propositions:
            if prop.is_set_relation:
                set_a = prop.subject_set
                set_b = prop.object_set
                
                if prop.relation == 'subset':
                    relations[(set_a, set_b)] = 'subset'
                elif prop.relation == 'superset':
                    relations[(set_a, set_b)] = 'superset'
                elif prop.relation == 'equal':
                    relations[(set_a, set_b)] = 'equal'
        
        return relations
```

---

## §2. 一致性維護

### §2.1 一致性監控器

```python
class ConsistencyMonitor:
    """
    一致性監控器
    
    持續監控決策過程中的一致性
    """
    
    def __init__(self):
        self.contradiction_detector = ContradictionDetector()
        self.consistency_history = []
        self.alert_thresholds = {
            'CRITICAL': 0,      # 任何關鍵矛盾都觸發警報
            'HIGH': 1,          # 1 個高嚴重性矛盾觸發警報
            'MEDIUM': 3,        # 3 個中等矛盾觸發警報
            'LOW': 5            # 5 個低嚴重性矛盾觸發警報
        }
    
    def monitor(self, decision: Decision) -> ConsistencyStatus:
        """
        監控決策的一致性狀態
        
        持續檢查：
        1. 決策前提的一致性
        2. 推理過程的一致性
        3. 結論之間的一致性
        4. 與歷史決策的一致性
        """
        
        status = ConsistencyStatus(decision_id=decision.id)
        
        # 1. 前提一致性
        premise_result = self.contradiction_detector.detect_contradictions(
            decision.premises, decision.context
        )
        status.premise_consistency = premise_result
        
        # 2. 推理過程一致性
        inference_result = self.contradiction_detector.detect_contradictions(
            self._extract_inference_conclusions(decision.proof_chain),
            decision.context
        )
        status.inference_consistency = inference_result
        
        # 3. 結論一致性
        conclusion_result = self.contradiction_detector.detect_contradictions(
            decision.conclusions, decision.context
        )
        status.conclusion_consistency = conclusion_result
        
        # 4. 歷史一致性
        historical_result = self._check_historical_consistency(decision)
        status.historical_consistency = historical_result
        
        # 5. 計算整體狀態
        status.overall = self._compute_overall_status([
            premise_result,
            inference_result,
            conclusion_result,
            historical_result
        ])
        
        # 6. 檢查是否觸發警報
        status.alerts = self._check_alerts(status.overall)
        
        # 7. 記錄歷史
        self._record_consistency(status)
        
        return status
    
    def _extract_inference_conclusions(self, proof_chain: ProofChain) -> Set[Proposition]:
        """
        從推理鏈中提取所有結論
        """
        
        conclusions = set()
        
        for step in proof_chain.steps:
            for conclusion in step.conclusions:
                conclusions.add(conclusion)
        
        return conclusions
    
    def _check_historical_consistency(self, decision: Decision) -> ContradictionResult:
        """
        檢查與歷史決策的一致性
        """
        
        contradictions = []
        
        # 獲取相關歷史決策
        relevant_decisions = self._get_relevant_historical_decisions(decision)
        
        for hist_decision in relevant_decisions:
            # 檢查前提衝突
            premise_conflict = self._check_premise_conflict(
                decision.premises, hist_decision.premises
            )
            if premise_conflict:
                contradictions.append(premise_conflict)
            
            # 檢查結論衝突
            conclusion_conflict = self._check_conclusion_conflict(
                decision.conclusions, hist_decision.conclusions
            )
            if conclusion_conflict:
                contradictions.append(conclusion_conflict)
        
        return ContradictionResult(
            has_contradiction=len(contradictions) > 0,
            contradictions=contradictions,
            count=len(contradictions),
            severity=self._calculate_severity(contradictions),
            resolution_suggestions=[]
        )
    
    def _compute_overall_status(self, results: List[ContradictionResult]) -> str:
        """
        計算整體一致性狀態
        """
        
        if not results:
            return 'UNKNOWN'
        
        has_any_contradiction = any(r.has_contradiction for r in results)
        
        if not has_any_contradiction:
            return 'CONSISTENT'
        
        # 根據最嚴重的矛盾確定狀態
        severities = [r.severity for r in results if r.has_contradiction]
        
        if 'CRITICAL' in severities:
            return 'CRITICAL_INCONSISTENT'
        elif 'HIGH' in severities:
            return 'HIGHLY_INCONSISTENT'
        elif 'MEDIUM' in severities:
            return 'MODERATELY_INCONSISTENT'
        else:
            return 'SLIGHTLY_INCONSISTENT'
    
    def _check_alerts(self, overall_status: str) -> List[Alert]:
        """
        檢查是否觸發警報
        """
        
        alerts = []
        
        alert_map = {
            'CRITICAL_INCONSISTENT': 'CRITICAL',
            'HIGHLY_INCONSISTENT': 'HIGH',
            'MODERATELY_INCONSISTENT': 'MEDIUM',
            'SLIGHTLY_INCONSISTENT': 'LOW'
        }
        
        if overall_status in alert_map:
            severity = alert_map[overall_status]
            threshold = self.alert_thresholds.get(severity, 0)
            
            # 這裡需要更詳細的實現
            alerts.append(Alert(
                severity=severity,
                message=f"一致性狀態：{overall_status}",
                threshold=threshold
            ))
        
        return alerts
    
    def _record_consistency(self, status: ConsistencyStatus):
        """
        記錄一致性狀態歷史
        """
        
        self.consistency_history.append({
            'timestamp': GetCurrentTimestamp(),
            'decision_id': status.decision_id,
            'overall': status.overall,
            'has_contradictions': status.overall != 'CONSISTENT'
        })
```

---

## §3. 衝突解決

### §3.1 衝突解決引擎

```python
class ConflictResolver:
    """
    衝突解決引擎
    
    自動解決檢測到的矛盾
    """
    
    def __init__(self):
        self.resolution_strategies = {
            'propositional': self._resolve_propositional,
            'set_theoretic': self._resolve_set_theoretic,
            'quantifier': self._resolve_quantifier,
            'contextual': self._resolve_contextual,
            'hierarchy': self._resolve_hierarchy
        }
    
    def resolve(self, contradiction: Contradiction, 
               context: DecisionContext) -> Resolution:
        """
        解決矛盾
        
        根據矛盾類型選擇適當的解決策略
        """
        
        resolver = self.resolution_strategies.get(contradiction.type)
        
        if not resolver:
            return Resolution(
                success=False,
                method='unknown',
                message=f"未知矛盾類型：{contradiction.type}"
            )
        
        return resolver(contradiction, context)
    
    def resolve_all(self, contradictions: List[Contradiction],
                   context: DecisionContext) -> ResolutionResult:
        """
        解決所有矛盾
        """
        
        resolutions = []
        
        for contradiction in contradictions:
            resolution = self.resolve(contradiction, context)
            resolutions.append(resolution)
        
        success_count = sum(1 for r in resolutions if r.success)
        
        return ResolutionResult(
            total=len(contradictions),
            successful=success_count,
            failed=len(contradictions) - success_count,
            resolutions=resolutions
        )
    
    def _resolve_propositional(self, contradiction: Contradiction,
                             context: DecisionContext) -> Resolution:
        """
        解決命題矛盾
        """
        
        # 策略 1：前提差異分析
        # 識別哪個前提可能是錯誤的
        
        # 策略 2：選擇更可靠的命題
        # 根據來源可靠性選擇保留哪個命題
        
        # 策略 3：引入新區分
        # 找到區分兩個命題的條件
        
        # 選擇策略
        if context.allow_context_separation:
            return Resolution(
                success=True,
                method='context-separation',
                message='通過語境分離解決矛盾',
                action='Separate contradictory propositions into different contexts'
            )
        
        elif context.has_preference:
            # 根據偏好選擇
            preferred = self._select_preferred_proposition(
                contradiction.proposition_a,
                contradiction.proposition_b,
                context.preference
            )
            
            return Resolution(
                success=True,
                method='preference-based',
                message=f'根據偏好選擇保留：{preferred}',
                action=f'Reject {preferred == contradiction.proposition_a and contradiction.proposition_b or contradiction.proposition_a}'
            )
        
        else:
            return Resolution(
                success=False,
                method='none-available',
                message='無法自動解決，需要人工介入',
                requires_human=True
            )
    
    def _resolve_set_theoretic(self, contradiction: Contradiction,
                               context: DecisionContext) -> Resolution:
        """
        解決集合矛盾
        """
        
        # 重新定義集合邊界
        return Resolution(
            success=True,
            method='boundary-redefinition',
            message='重新定義集合邊界以解決循環包含',
            action='Redefine set boundaries to eliminate circular subset relationship'
        )
    
    def _resolve_quantifier(self, contradiction: Contradiction,
                           context: DecisionContext) -> Resolution:
        """
        解決量化矛盾
        """
        
        # 縮小存在量詞或全稱量詞的範圍
        return Resolution(
            success=True,
            method='scope-reduction',
            message='縮小量詞範圍以解決矛盾',
            action='Reduce quantifier scope to resolve contradiction'
        )
    
    def _resolve_contextual(self, contradiction: Contradiction,
                           context: DecisionContext) -> Resolution:
        """
        解決語境矛盾
        """
        
        # 明確區分不同語境的適用範圍
        return Resolution(
            success=True,
            method='context-clarification',
            message='明確語境邊界',
            action='Clarify context boundaries to prevent overlap'
        )
    
    def _resolve_hierarchy(self, contradiction: Contradiction,
                         context: DecisionContext) -> Resolution:
        """
        解決層級矛盾
        """
        
        # 根據 SA-L 層級解決
        # 上位層級的決策優先
        
        if contradiction.sa_level_a < contradiction.sa_level_b:
            # A 優先
            return Resolution(
                success=True,
                method='hierarchy-priority',
                message=f'SA-L{contradiction.sa_level_a} 優先於 SA-L{contradiction.sa_level_b}',
                action=f'Accept SA-L{contradiction.sa_level_a} decision, reject SA-L{contradiction.sa_level_b}'
            )
        else:
            return Resolution(
                success=True,
                method='hierarchy-priority',
                message=f'SA-L{contradiction.sa_level_b} 優先於 SA-L{contradiction.sa_level_a}',
                action=f'Accept SA-L{contradiction.sa_level_b} decision, reject SA-L{contradiction.sa_level_a}'
            )
```

### §3.2 衝突仲裁

```python
class ConflictArbiter:
    """
    衝突仲裁器
    
    當多個解決方案衝突時進行仲裁
    """
    
    def __init__(self):
        self.preference_weights = {
            'survival': 100,      # 生存優先
            'consistency': 90,    # 一致性
            'completeness': 80,   # 完整性
            'efficiency': 70,     # 效率
            'simplicity': 60      # 簡單性
        }
    
    def arbitrate(self, resolutions: List[Resolution],
                 context: DecisionContext) -> Resolution:
        """
        仲裁多個解決方案
        
        選擇最優解決方案
        """
        
        if len(resolutions) == 1:
            return resolutions[0]
        
        # 評分每個解決方案
        scored_resolutions = []
        
        for resolution in resolutions:
            score = self._score_resolution(resolution, context)
            scored_resolutions.append((score, resolution))
        
        # 選擇最高分的解決方案
        scored_resolutions.sort(key=lambda x: x[0], reverse=True)
        
        return scored_resolutions[0][1]
    
    def _score_resolution(self, resolution: Resolution,
                        context: DecisionContext) -> float:
        """
        評分解決方案
        """
        
        score = 0.0
        
        # 成功與否
        if resolution.success:
            score += 50
        
        # 是否需要人工介入
        if not resolution.requires_human:
            score += 30
        
        # 與上下文偏好的一致性
        if context.preferred_strategy:
            if resolution.method == context.preferred_strategy:
                score += 20
        
        # 計算複雜度 penalty
        if resolution.action and len(resolution.action) > 100:
            score -= 10
        
        return score
```

---

## §4. 審計與報告

### §4.1 一致性審計日誌

```python
def LogConsistencyCheckToAuditTrail(check_result, decision):
    """
    將一致性檢查結果記錄至 AUDIT_TRAIL
    """
    
    audit_entry = {
        'event': 'CONSISTENCY_CHECK',
        'timestamp': GetCurrentTimestamp(),
        
        # 決策識別
        'decision_id': decision.id,
        'sa_level': decision.sa_level,
        
        # 一致性狀態
        'overall_status': check_result.overall_status,
        
        # 各層面一致性
        'premise_consistent': not check_result.premise_consistency.has_contradiction,
        'inference_consistent': not check_result.inference_consistency.has_contradiction,
        'conclusion_consistent': not check_result.conclusion_consistency.has_contradiction,
        'historical_consistent': not check_result.historical_consistency.has_contradiction,
        
        # 矛盾數量
        'total_contradictions': sum([
            check_result.premise_consistency.count,
            check_result.inference_consistency.count,
            check_result.conclusion_consistency.count,
            check_result.historical_consistency.count
        ]),
        
        # 嚴重性
        'severity': check_result.overall.get('severity', 'UNKNOWN'),
        
        # 警報
        'alerts_triggered': len(check_result.alerts)
    }
    
    AppendToAuditTrail(audit_entry)
    
    return audit_entry


def LogConflictResolutionToAuditTrail(resolution_result, contradictions):
    """
    將衝突解決結果記錄至 AUDIT_TRAIL
    """
    
    audit_entry = {
        'event': 'CONFLICT_RESOLUTION',
        'timestamp': GetCurrentTimestamp(),
        
        # 解決結果
        'total_conflicts': resolution_result.total,
        'resolved_successfully': resolution_result.successful,
        'resolution_failed': resolution_result.failed,
        
        # 解決方法統計
        'methods_used': list(set(r.method for r in resolution_result.resolutions)),
        
        # 每個矛盾的詳細解決信息
        'resolution_details': [
            {
                'contradiction_type': c.type,
                'resolution_method': r.method,
                'success': r.success,
                'requires_human': r.requires_human
            }
            for c, r in zip(contradictions, resolution_result.resolutions)
        ]
    }
    
    AppendToAuditTrail(audit_entry)
    
    return audit_entry
```

### §4.2 一致性報告生成

```python
def generate_consistency_report(consistency_status: ConsistencyStatus) -> str:
    """
    生成一致性報告
    """
    
    report = f"""
================================================================================
                    一致性驗證報告
================================================================================

決策 ID: {consistency_status.decision_id}
生成時間: {consistency_status.timestamp}

--------------------------------------------------------------------------------
整體狀態
--------------------------------------------------------------------------------
{consistency_status.overall}

--------------------------------------------------------------------------------
各層面一致性
--------------------------------------------------------------------------------
前提一致性:    {'✓ 一致' if consistency_status.premise_consistency.has_contradiction else '✗ 矛盾'}
              矛盾數量: {consistency_status.premise_consistency.count}

推理一致性:    {'✓ 一致' if consistency_status.inference_consistency.has_contradiction else '✗ 矛盾'}
              矛盾數量: {consistency_status.inference_consistency.count}

結論一致性:    {'✓ 一致' if consistency_status.conclusion_consistency.has_contradiction else '✗ 矛盾'}
              矛盾數量: {consistency_status.conclusion_consistency.count}

歷史一致性:    {'✓ 一致' if consistency_status.historical_consistency.has_contradiction else '✗ 矛盾'}
              矛盾數量: {consistency_status.historical_consistency.count}

--------------------------------------------------------------------------------
警報
--------------------------------------------------------------------------------
"""
    
    if consistency_status.alerts:
        for alert in consistency_status.alerts:
            report += f"- [{alert.severity}] {alert.message}\n"
    else:
        report += "無警報觸發\n"
    
    report += """
================================================================================
"""
    
    return report
```

---

## §5. 版本與演化

| 版本 | 日期 | 變更摘要 |
|------|------|----------|
| v2.2 | 2026-03 | 初始版本，建立一致性驗證框架 |

---

*NoieLogicAGENTS — CONSISTENCY_ENGINE 模組*  
*Logic-OS v2.2 一致性驗證核心*  
*矛盾偵測、一致性維護、衝突解決*
