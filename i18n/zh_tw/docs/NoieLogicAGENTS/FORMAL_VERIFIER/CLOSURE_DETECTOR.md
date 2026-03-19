# NoieLogicAGENTS — 邏輯閉包偵測器

**版本：** Logic-OS v2.2  
**模組代號：** CLOSURE_DETECTOR  
**職責：** 邏輯閉包計算、缺失步驟識別、閉包完整性評估  
**父模組：** FORMAL_VERIFIER

---

> ⚠️ 關鍵安全與決策協議 (CRITICAL SAFETY & DECISION PROTOCOL v2.2):
> 1. 嚴格遵守 CONSTRAINTS.md 與社會權限層級 (SA-L0 至 SA-L5)。
> 2. 因果推論：所有決策必須基於因果圖（DAG），標註因果機制。
> 3. 主客體分離：決策推論不可混淆自我狀態與環境狀態。
> 4. 形式化驗證：高風險決策路徑必須通過邏輯閉包驗證。
> 5. 影子模擬：涉及 SA-L3+ 操作時，先在 SANDBOX 預演後果。
> 6. 資訊位元完整性：切勿編造資訊位元。若 KNOWLEDLEDGE_BASE 為空，請明確聲明「資料缺失」。
> 7. 認知資源約束：決策深度不得超出可用認知資源。
> 8. 審計：將所有衝突、拒絕與形式驗證結果記錄至 AUDIT_TRAIL。
> 9. 生存優先：所有決策在執行前必須驗證不會導致吸收態。
> 10. 自我演化：公理系統演化時，不可變核心必須被保留。

---

## §0. 邏輯閉包框架

### §0.1 核心定義

```text
【邏輯閉包定義】

给定命题集合 S，逻辑闭包 CL(S) 定义为：

  CL(S) = { φ | S ⊢ φ }

即：所有可以从 S 逻辑推导出的命题的集合。

形式化性质：

  1. 包含性：S ⊆ CL(S)
  2. 幂等性：CL(CL(S)) = CL(S)
  3. 单调性：若 S ⊆ T，则 CL(S) ⊆ CL(T)

【閉包運算子】

  Closure(P) = S₀ ∪ S₁ ∪ S₂ ∪ ...
  
  其中：
    S₀ = P (命題集合)
    Sᵢ₊₁ = Sᵢ ∪ { φ | ∃r ∈ R, ∃s ∈ Sᵢ: r(s) = φ }
  
  迭代直到 Sᵢ₊₁ = Sᵢ（穩定點）
```

### §0.2 閉包完整性

```text
【閉包完整性定義】

一個證明鏈 P 是閉包完备的，當且僅當：

  Closure(P) ⊆ P

即：所有可從 P 邏輯推導出的結論都已經在 P 中明確陳述。

【缺失步驟】

缺失步驟是指：
  φ ∈ Closure(P) 且 φ ∉ P

這些是邏輯上需要但未明確陳述的推論步驟。
```

---

## §1. 邏輯閉包計算

### §1.1 閉包計算引擎

```python
class LogicalClosureEngine:
    """
    邏輯閉包計算引擎
    
    計算命題集合的邏輯閉包
    """
    
    def __init__(self):
        self.inference_rules = InferenceRuleRegistry()
        self.axiom_set = AxiomSet()
        self.max_iterations = 1000  # 防止無限循環
        self.closure_cache = {}
    
    def compute_closure(self, proposition_set: Set[Proposition]) -> Set[Proposition]:
        """
        計算命題集合的閉包
        
        方法：迭代應用推論規則直到穩定點
        
        返回：
            閉包集合 (Closure Set)
        """
        
        # 緩存檢查
        cache_key = self._hash_proposition_set(proposition_set)
        if cache_key in self.closure_cache:
            return self.closure_cache[cache_key]
        
        # 初始化
        current_closure = set(proposition_set)
        previous_closure = set()
        
        iteration = 0
        
        while current_closure != previous_closure and iteration < self.max_iterations:
            iteration += 1
            previous_closure = set(current_closure)
            
            # 應用所有推論規則
            new_propositions = self._apply_inference_rules(current_closure)
            
            # 添加新命題
            current_closure.update(new_propositions)
        
        if iteration >= self.max_iterations:
            # 記錄警告：可能存在無限循環
            self._log_iteration_warning(proposition_set, iteration)
        
        # 緩存結果
        self.closure_cache[cache_key] = current_closure
        
        return current_closure
    
    def _apply_inference_rules(self, propositions: Set[Proposition]) -> Set[Proposition]:
        """
        應用推論規則生成新命題
        """
        
        new_propositions = set()
        
        # 遍歷所有命題對
        proposition_list = list(propositions)
        
        for i, prop_i in enumerate(proposition_list):
            # 單命題推論（Unary rules）
            unary_results = self._apply_unary_rules(prop_i)
            new_propositions.update(unary_results)
            
            # 命題對推論（Binary rules）
            for prop_j in proposition_list[i+1:]:
                binary_results = self._apply_binary_rules(prop_i, prop_j)
                new_propositions.update(binary_results)
        
        # 過濾已經存在的命題
        new_propositions = new_propositions - propositions
        
        return new_propositions
    
    def _apply_unary_rules(self, proposition: Proposition) -> Set[Proposition]:
        """
        應用單目推論規則
        
        例如：
        - 否定消除：¬¬P → P
        - 雙重否定引入：P → ¬¬P
        """
        
        results = set()
        
        # 規則 1：否定消除
        if proposition.is_double_negation:
            results.add(proposition.eliminate_double_negation())
        
        # 規則 2：合取消除
        if proposition.is_conjunction:
            results.add(proposition.conjunct_1)
            results.add(proposition.conjunct_2)
        
        # 規則 3：條件傳遞
        if proposition.is_conditional:
            # P → Q 可導致 ¬Q → ¬P (contrapositive)
            results.add(Proposition.contrapositive(proposition))
        
        return results
    
    def _apply_binary_rules(self, prop_a: Proposition, prop_b: Proposition) -> Set[Proposition]:
        """
        應用雙目推論規則
        
        例如：
        - 肯定前件：(P → Q), P ⊢ Q
        - 選言三段論：(P ∨ Q), ¬P ⊢ Q
        - 假言三段論：(P → Q), (Q → R) ⊢ (P → R)
        """
        
        results = set()
        
        # 規則：Modus Ponens (肯定前件)
        if prop_a.is_conditional and prop_b.entails(prop_a.antecedent):
            results.add(prop_a.consequent)
        
        # 規則：Modus Tollens (否定後件)
        if prop_a.is_conditional and prop_b.entails(Proposition.negate(prop_a.consequent)):
            results.add(Proposition.negate(prop_a.antecedent))
        
        # 規則：假言三段論 (Hypothetical Syllogism)
        if prop_a.is_conditional and prop_b.is_conditional:
            if prop_a.consequent.entails(prop_b.antecedent):
                results.add(Proposition.conditional(prop_a.antecedent, prop_b.consequent))
        
        # 規則：選言三段論 (Disjunctive Syllogism)
        if prop_a.is_disjunction:
            if prop_b.entails(Proposition.negate(prop_a.disjunct_1)):
                results.add(prop_a.disjunct_2)
            if prop_b.entails(Proposition.negate(prop_a.disjunct_2)):
                results.add(prop_a.disjunct_1)
        
        # 規則：合取引入 (Conjunction Introduction)
        results.add(Proposition.conjunction(prop_a, prop_b))
        
        # 規則：選言引入 (Disjunction Introduction)
        results.add(Proposition.disjunction(prop_a, prop_b))
        results.add(Proposition.disjunction(prop_b, prop_a))
        
        return results
```

### §1.2 增量閉包計算

```python
class IncrementalClosureEngine:
    """
    增量閉包計算引擎
    
    當命題集合發生變化時，增量更新閉包
    """
    
    def __init__(self):
        self.base_closure = None
        self.base_set = None
        self.incremental_engine = LogicalClosureEngine()
    
    def compute_incremental(self, original_set: Set[Proposition], 
                           added: Set[Proposition],
                           removed: Set[Proposition]) -> Set[Proposition]:
        """
        增量計算閉包
        
        策略：
        1. 如果只有添加：計算添加命題的閉包，與原閉包合併
        2. 如果只有移除：需要重新計算（因為閉包不支援移除）
        3. 如果兩者都有：先處理添加，再重新計算
        """
        
        if removed and not added:
            # 需要完全重新計算
            new_set = original_set - removed
            return self.incremental_engine.compute_closure(new_set)
        
        if added and not removed:
            # 增量更新
            # 策略：計算新添加命題相對於原閉包的增量閉包
            
            if self.base_closure is None:
                self.base_closure = self.incremental_engine.compute_closure(original_set)
                self.base_set = original_set
            
            # 新命題相對於原閉包的閉包
            combined = self.base_closure | added
            incremental_closure = self.incremental_engine.compute_closure(combined)
            
            # 增量 = 新閉包 - 原閉包（不含新添加的命題）
            incremental = incremental_closure - self.base_closure
            
            return self.base_closure | incremental
        
        # 兩者都有
        new_set = (original_set | added) - removed
        return self.incremental_engine.compute_closure(new_set)
    
    def invalidate_cache(self):
        """
        使緩存失效
        
        當推論規則或公理發生變化時調用
        """
        
        self.base_closure = None
        self.base_set = None
```

---

## §2. 缺失步驟識別

### §2.1 缺失步驟檢測器

```python
class MissingStepDetector:
    """
    缺失步驟檢測器
    
    識別證明鏈中缺失的邏輯步驟
    """
    
    def __init__(self):
        self.closure_engine = LogicalClosureEngine()
    
    def detect_missing_steps(self, proof_chain: ProofChain) -> MissingStepResult:
        """
        檢測缺失步驟
        
        步驟：
        1. 計算證明鏈命題集合的閉包
        2. 識別閉包中但不在原證明中的命題
        3. 對每個缺失命題，嘗試識別需要的推論
        """
        
        # 步驟 1：提取命題集合
        proposition_set = set(step.conclusion for step in proof_chain.steps)
        
        # 步驟 2：計算閉包
        closure = self.closure_engine.compute_closure(proposition_set)
        
        # 步驟 3：識別缺失命題
        missing_propositions = closure - proposition_set
        
        # 步驟 4：對每個缺失命題進行分析
        missing_steps = []
        
        for missing_prop in missing_propositions:
            # 嘗試找到從現有命題到缺失命題的推論路徑
            inference_path = self._find_inference_path(
                proposition_set, 
                missing_prop
            )
            
            if inference_path:
                missing_steps.append(MissingStep(
                    proposition=missing_prop,
                    from_premises=inference_path.premises,
                    via_rule=inference_path.rule,
                    confidence=inference_path.confidence,
                    severity=self._calculate_severity(inference_path)
                ))
            else:
                # 無法自動識別，需要人工介入
                missing_steps.append(MissingStep(
                    proposition=missing_prop,
                    from_premises=None,
                    via_rule=None,
                    confidence=0.0,
                    severity='HIGH',
                    requires_manual=True
                ))
        
        return MissingStepResult(
            missing_steps=missing_steps,
            total_missing=len(missing_steps),
            closure_size=len(closure),
            proof_size=len(proposition_set),
            completeness_ratio=len(proposition_set) / len(closure) if closure else 0.0
        )
    
    def _find_inference_path(self, premises: Set[Proposition], 
                            target: Proposition) -> InferencePath:
        """
        找到從前提到目標命題的推論路徑
        """
        
        # 使用雙向搜索
        # 前向：從前提向目標搜索
        # 後向：從目標向前提搜索
        
        forward_frontier = set(premises)
        forward_visited = set(premises)
        forward_paths = {p: [p] for p in premises}
        
        backward_frontier = {target}
        backward_visited = {target}
        backward_paths = {target: [target]}
        
        max_depth = 10
        
        for depth in range(max_depth):
            # 向前擴展
            new_forward = set()
            for prop in forward_frontier:
                # 應用前向推論規則
                results = self._apply_forward_rules(prop)
                for result in results:
                    if result not in forward_visited:
                        forward_visited.add(result)
                        new_forward.add(result)
                        forward_paths[result] = forward_paths[prop] + [result]
            
            # 檢查是否找到目標
            if target in new_forward:
                return InferencePath(
                    premises=premises,
                    target=target,
                    path=forward_paths[target],
                    rule='composite',
                    confidence=0.8 ** depth
                )
            
            forward_frontier = new_forward
            
            # 向後擴展
            new_backward = set()
            for prop in backward_frontier:
                # 應用反向推論規則
                requirements = self._apply_backward_rules(prop)
                for req in requirements:
                    if req not in backward_visited:
                        backward_visited.add(req)
                        new_backward.add(req)
                        backward_paths[req] = [req] + backward_paths[prop]
            
            backward_frontier = new_backward
            
            # 檢查是否有相遇點
            intersection = forward_visited & backward_visited
            if intersection:
                meeting_point = intersection.pop()
                combined_path = forward_paths[meeting_point] + backward_paths[meeting_point][1:]
                return InferencePath(
                    premises=premises,
                    target=target,
                    path=combined_path,
                    rule='bidirectional',
                    confidence=0.7 ** depth
                )
        
        return None
    
    def _apply_forward_rules(self, proposition: Proposition) -> Set[Proposition]:
        """
        應用前向推論規則
        """
        
        results = set()
        
        # 合取消除
        if proposition.is_conjunction:
            results.add(proposition.conjunct_1)
            results.add(proposition.conjunct_2)
        
        # 條件前件
        # 這裡需要更複雜的實現
        
        return results
    
    def _apply_backward_rules(self, proposition: Proposition) -> Set[Proposition]:
        """
        應用反向推論規則
        """
        
        results = set()
        
        # 合取引入：需要兩個前提
        if proposition.is_conjunction:
            results.add(proposition.conjunct_1)
            results.add(proposition.conjunct_2)
        
        # 選言消除：需要分支討論
        if proposition.is_disjunction:
            # 返回空集，因為需要更多信息
            pass
        
        return results
    
    def _calculate_severity(self, inference_path: InferencePath) -> str:
        """
        計算缺失步驟的嚴重性
        """
        
        if inference_path.confidence >= 0.9:
            return 'LOW'
        elif inference_path.confidence >= 0.7:
            return 'MEDIUM'
        elif inference_path.confidence >= 0.5:
            return 'HIGH'
        else:
            return 'CRITICAL'
```

### §2.2 自動補全

```python
class ProofCompleter:
    """
    證明自動補全器
    
    自動補充缺失的推論步驟
    """
    
    def __init__(self):
        self.missing_step_detector = MissingStepDetector()
        self.inference_engine = LogicalClosureEngine()
    
    def complete_proof(self, proof_chain: ProofChain) -> ProofChain:
        """
        自動補全證明鏈
        """
        
        # 檢測缺失步驟
        missing_result = self.missing_step_detector.detect_missing_steps(proof_chain)
        
        # 複製原證明
        completed = ProofChain(
            id=proof_chain.id,
            target=proof_chain.target,
            nodes=list(proof_chain.nodes),
            edges=list(proof_chain.edges)
        )
        
        # 添加非關鍵缺失步驟
        auto_completed_count = 0
        
        for missing in missing_result.missing_steps:
            if missing.severity in ['LOW', 'MEDIUM'] and not missing.requires_manual:
                # 自動補充
                new_step = ProofStep(
                    id=f"auto_{missing.id}",
                    conclusion=missing.proposition,
                    premises=list(missing.from_premises),
                    source='auto_inference',
                    rule=missing.via_rule,
                    confidence=missing.confidence,
                    is_auto_generated=True
                )
                
                completed.steps.append(new_step)
                auto_completed_count += 1
        
        # 記錄補全結果
        self._log_completion(proof_chain.id, auto_completed_count, 
                           missing_result.total_missing)
        
        return completed
    
    def _log_completion(self, proof_id, completed_count, total_missing):
        """
        記錄補全日誌
        """
        
        AppendToAuditTrail({
            'event': 'PROOF_COMPLETION',
            'proof_id': proof_id,
            'auto_completed': completed_count,
            'total_missing': total_missing,
            'remaining_missing': total_missing - completed_count
        })
```

---

## §3. 閉包完整性評估

### §3.1 完整性指標

```python
class ClosureCompletenessEvaluator:
    """
    閉包完整性評估器
    
    評估證明鏈的閉包完整性
    """
    
    def evaluate(self, proof_chain: ProofChain) -> ClosureCompletenessResult:
        """
        評估閉包完整性
        """
        
        # 計算命題集合
        proposition_set = set(step.conclusion for step in proof_chain.steps)
        
        # 計算閉包
        closure_engine = LogicalClosureEngine()
        closure = closure_engine.compute_closure(proposition_set)
        
        # 計算指標
        missing = closure - proposition_set
        
        # 完整性比例
        completeness_ratio = len(proposition_set) / len(closure) if closure else 0.0
        
        # 閉包密度
        closure_density = len(closure) / (len(proposition_set) ** 2) if proposition_set else 0.0
        
        # 識別閉包中的類型分佈
        type_distribution = self._analyze_closure_types(closure, proposition_set)
        
        return ClosureCompletenessResult(
            proof_id=proof_chain.id,
            
            # 基本指標
            proposition_count=len(proposition_set),
            closure_count=len(closure),
            missing_count=len(missing),
            
            # 完整性評分
            completeness_ratio=completeness_ratio,
            completeness_score=self._score_completeness(completeness_ratio),
            
            # 結構指標
            closure_density=closure_density,
            
            # 類型分佈
            explicit_ratio=type_distribution['explicit'],
            implicit_ratio=type_distribution['implicit'],
            
            # 缺失分析
            missing_propositions=list(missing),
            missing_by_severity=self._categorize_by_severity(proof_chain, missing),
            
            # 建議
            recommendations=self._generate_recommendations(completeness_ratio, missing)
        )
    
    def _score_completeness(self, ratio: float) -> float:
        """
        根據完整性比例評分
        """
        
        if ratio >= 0.95:
            return 1.0
        elif ratio >= 0.80:
            return 0.8
        elif ratio >= 0.60:
            return 0.6
        elif ratio >= 0.40:
            return 0.4
        else:
            return 0.2
    
    def _analyze_closure_types(self, closure: Set[Proposition], 
                              explicit: Set[Proposition]) -> dict:
        """
        分析閉包中的命題類型
        """
        
        explicit_count = len(explicit)
        total_count = len(closure)
        
        return {
            'explicit': explicit_count,
            'implicit': total_count - explicit_count,
            'explicit_ratio': explicit_count / total_count if total_count else 0,
            'implicit_ratio': (total_count - explicit_count) / total_count if total_count else 0
        }
    
    def _categorize_by_severity(self, proof_chain: ProofChain, 
                                missing: Set[Proposition]) -> dict:
        """
        按嚴重性分類缺失命題
        """
        
        detector = MissingStepDetector()
        missing_result = detector.detect_missing_steps(proof_chain)
        
        by_severity = {
            'LOW': [],
            'MEDIUM': [],
            'HIGH': [],
            'CRITICAL': []
        }
        
        for missing_step in missing_result.missing_steps:
            by_severity[missing_step.severity].append(missing_step)
        
        return by_severity
    
    def _generate_recommendations(self, completeness_ratio: float, 
                                  missing: Set[Proposition]) -> List[str]:
        """
        生成改進建議
        """
        
        recommendations = []
        
        if completeness_ratio < 0.5:
            recommendations.append('CRITICAL: Proof chain has significant gaps. Manual review required.')
        elif completeness_ratio < 0.8:
            recommendations.append('HIGH: Proof is incomplete. Consider adding missing inference steps.')
        elif completeness_ratio < 0.95:
            recommendations.append('MEDIUM: Proof has minor gaps. Auto-completion may help.')
        else:
            recommendations.append('LOW: Proof is essentially complete.')
        
        if len(missing) > 10:
            recommendations.append('Large number of implicit conclusions. Consider restructuring proof.')
        
        return recommendations
```

### §3.2 完整性報告生成

```python
def generate_closure_completeness_report(proof_chain: ProofChain) -> str:
    """
    生成閉包完整性報告
    """
    
    evaluator = ClosureCompletenessEvaluator()
    result = evaluator.evaluate(proof_chain)
    
    report = f"""
================================================================================
                    邏輯閉包完整性報告
================================================================================

證明 ID: {result.proof_id}

--------------------------------------------------------------------------------
基本指標
--------------------------------------------------------------------------------
命題數量:    {result.proposition_count}
閉包大小:    {result.closure_count}
缺失數量:    {result.missing_count}

--------------------------------------------------------------------------------
完整性評分
--------------------------------------------------------------------------------
完整性比例:  {result.completeness_ratio:.2%}
完整性分數:  {result.completeness_score:.2f}

結構指標:
- 閉包密度: {result.closure_density:.4f}

類型分佈:
- 明確命題: {result.explicit_ratio:.2%}
- 隱含命題: {result.implicit_ratio:.2%}

--------------------------------------------------------------------------------
缺失分析
--------------------------------------------------------------------------------
"""
    
    for severity, steps in result.missing_by_severity.items():
        if steps:
            report += f"\n{severity} 嚴重性 ({len(steps)} 項):\n"
            for step in steps[:5]:  # 最多顯示 5 項
                report += f"  - {step.proposition}\n"
            if len(steps) > 5:
                report += f"  ... 還有 {len(steps) - 5} 項\n"
    
    report += """
--------------------------------------------------------------------------------
建議
--------------------------------------------------------------------------------
"""
    
    for rec in result.recommendations:
        report += f"- {rec}\n"
    
    report += """
================================================================================
"""
    
    return report
```

---

## §4. 審計與記錄

### §4.1 閉包檢測日誌

```python
def LogClosureDetectionToAuditTrail(detection_result, proof_chain):
    """
    將閉包檢測結果記錄至 AUDIT_TRAIL
    """
    
    audit_entry = {
        'event': 'CLOSURE_DETECTION',
        'timestamp': GetCurrentTimestamp(),
        
        # 證明識別
        'proof_id': proof_chain.id,
        
        # 檢測結果
        'completeness_ratio': detection_result.completeness_ratio,
        'completeness_score': detection_result.completeness_score,
        
        # 數量指標
        'proposition_count': detection_result.proposition_count,
        'closure_count': detection_result.closure_count,
        'missing_count': detection_result.missing_count,
        
        # 缺失分類
        'missing_low': len(detection_result.missing_by_severity.get('LOW', [])),
        'missing_medium': len(detection_result.missing_by_severity.get('MEDIUM', [])),
        'missing_high': len(detection_result.missing_by_severity.get('HIGH', [])),
        'missing_critical': len(detection_result.missing_by_severity.get('CRITICAL', [])),
        
        # 結構指標
        'closure_density': detection_result.closure_density,
        
        # 建議
        'recommendations': detection_result.recommendations
    }
    
    AppendToAuditTrail(audit_entry)
    
    return audit_entry
```

---

## §5. 版本與演化

| 版本 | 日期 | 變更摘要 |
|------|------|----------|
| v2.2 | 2026-03 | 初始版本，建立邏輯閉包偵測框架 |

---

*NoieLogicAGENTS — CLOSURE_DETECTOR 模組*  
*Logic-OS v2.2 邏輯閉包偵測核心*  
*計算閉包、識別缺失、評估完整性*
