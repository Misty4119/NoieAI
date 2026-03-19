# NoieLogicAGENTS — 證明鏈檢查器

**版本：** Logic-OS v2.2  
**模組代號：** PROOF_CHECKER  
**職責：** 驗證證明有效性、建構證明鏈、追溯公理/引理/定理來源  
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

## §0. 證明鏈檢查框架

### §0.1 核心定義

```text
【證明鏈定義】

一個證明鏈 P 是有效的，當且僅當：

 1. 結構完整性：P = (S₁, S₂, ..., Sₙ)，每個 Sᵢ 是邏輯陳述
 2. 推論有效性：每對相鄰步驟 (Sᵢ, Sᵢ₊₁) 滿足有效推論規則
 3. 源可追溯：每個 Sᵢ 可追溯至公理、引理或因果圖
 4. 無跳躍：不存在邏輯跳躍（missing link）
 5. 結論支撐：最終結論 C 被完整支撐

形式化表示：

  ValidProof(P) ⟺ 
    Structure(P) ∧ 
    InferenceValid(P) ∧ 
    SourceTraceable(P) ∧ 
    NoGaps(P) ∧ 
    ConclusionSupported(P, C)
```

### §0.2 證明元素分類

| 元素類型 | 符號表示 | 信心度 | 驗證要求 |
|----------|----------|--------|----------|
| **公理 (Axiom)** | A ∈ Θ | 1.0 | 不可變核心成員 |
| **定義 (Definition)** | D ∈ Δ | 1.0 | 語義一致性 |
| **定理 (Theorem)** | T ∈ Τ | ≥ 0.99 | 完整形式證明 |
| **引理 (Lemma)** | L ∈ Λ | ≥ 0.95 | 局部有效性 |
| **推論 (Inference)** | I ∈ Ι | ≥ 0.80 | 因果圖支撐 |
| **假設 (Assumption)** | H ∈ Η | ≥ 0.50 | 明確標記 |

---

## §1. 證明有效性驗證

### §1.1 結構驗證算法

```python
class ProofStructureValidator:
    """
    證明結構驗證器
    
    檢查證明鏈的結構完整性
    """
    
    def __init__(self):
        self.axiom_set = self._load_axioms()
        self.inference_rules = self._load_inference_rules()
    
    def validate_structure(self, proof_chain: ProofChain) -> StructureValidationResult:
        """
        驗證證明結構
        
        檢查項目：
        1. 節點完整性：每個節點都有結論
        2. 邊有效性：每條邊都是有效推論
        3. 連通性：從前提到結論存在路徑
        4. 無環路：不存在循環論證
        """
        
        issues = []
        
        # 檢查 1：節點完整性
        for node in proof_chain.nodes:
            if not node.conclusion:
                issues.append({
                    'type': 'MISSING_CONCLUSION',
                    'node_id': node.id,
                    'severity': 'HIGH'
                })
        
        # 檢查 2：邊有效性
        for edge in proof_chain.edges:
            if not self._is_valid_inference(edge.premise, edge.conclusion):
                issues.append({
                    'type': 'INVALID_INFERENCE',
                    'edge_id': edge.id,
                    'premise': edge.premise,
                    'conclusion': edge.conclusion,
                    'severity': 'HIGH'
                })
        
        # 檢查 3：連通性
        if not self._is_connected(proof_chain):
            issues.append({
                'type': 'DISCONNECTED_PROOF',
                'severity': 'HIGH'
            })
        
        # 檢查 4：無環路
        if self._has_cycle(proof_chain):
            issues.append({
                'type': 'CIRCULAR_REASONING',
                'severity': 'CRITICAL'
            })
        
        return StructureValidationResult(
            valid=len([i for i in issues if i['severity'] == 'HIGH']) == 0,
            issues=issues,
            node_count=len(proof_chain.nodes),
            edge_count=len(proof_chain.edges)
        )
    
    def _is_valid_inference(self, premise: Statement, conclusion: Statement) -> bool:
        """
        檢查推論是否有效
        
        有效推論規則：
        - 演繹 (Deduction): ∀x(P(x) → Q(x)), P(a) ⊢ Q(a)
        - 歸納 (Induction): P(a), P(b), ... ⊢ ∀xP(x)
        - 溯因 (Abduction): Q(a), P(a) → Q(a) ⊢ P(a)
        - 類比 (Analogy): P(a), Q(a), P(b) → Q(b) ⊢ P(b) → Q(b)
        """
        
        rule = self.inference_rules.match(premise, conclusion)
        
        if not rule:
            return False
        
        return rule.is_valid(premise, conclusion)
    
    def _has_cycle(self, proof_chain: ProofChain) -> bool:
        """
        檢測循環論證
        
        使用深度優先搜索 (DFS) 檢測環路
        """
        
        visited = set()
        rec_stack = set()
        
        def dfs(node_id: str) -> bool:
            visited.add(node_id)
            rec_stack.add(node_id)
            
            for neighbor in proof_chain.get_successors(node_id):
                if neighbor not in visited:
                    if dfs(neighbor):
                        return True
                elif neighbor in rec_stack:
                    return True
            
            rec_stack.remove(node_id)
            return False
        
        for node in proof_chain.nodes:
            if node.id not in visited:
                if dfs(node.id):
                    return True
        
        return False
```

### §1.2 推論規則庫

```python
class InferenceRuleRegistry:
    """
    推論規則註冊表
    
    管理所有有效的邏輯推論規則
    """
    
    RULES = {
        'modus_ponens': {
            'name': '肯定前件 (Modus Ponens)',
            'pattern': '(P → Q), P ⊢ Q',
            'formal': '∀x(P(x) → Q(x)), P(a) ⊢ Q(a)',
            'validity': 1.0,
            'category': 'deductive'
        },
        
        'modus_tollens': {
            'name': '否定後件 (Modus Tollens)',
            'pattern': '(P → Q), ¬Q ⊢ ¬P',
            'formal': '∀x(P(x) → Q(x)), ¬Q(a) ⊢ ¬P(a)',
            'validity': 1.0,
            'category': 'deductive'
        },
        
        'hypothetical_syllogism': {
            'name': '假言三段論',
            'pattern': '(P → Q), (Q → R) ⊢ (P → R)',
            'validity': 1.0,
            'category': 'deductive'
        },
        
        'disjunctive_syllogism': {
            'name': '選言三段論',
            'pattern': '(P ∨ Q), ¬P ⊢ Q',
            'validity': 1.0,
            'category': 'deductive'
        },
        
        'conjunction_intro': {
            'name': '合取引入',
            'pattern': 'P, Q ⊢ (P ∧ Q)',
            'validity': 1.0,
            'category': 'deductive'
        },
        
        'conjunction_elim': {
            'name': '合取消除',
            'pattern': '(P ∧ Q) ⊢ P',
            'validity': 1.0,
            'category': 'deductive'
        },
        
        'disjunction_intro': {
            'name': '選言引入',
            'pattern': 'P ⊢ (P ∨ Q)',
            'validity': 1.0,
            'category': 'deductive'
        },
        
        'universal_instantiation': {
            'name': '全稱例化',
            'pattern': '∀xP(x) ⊢ P(a)',
            'validity': 1.0,
            'category': 'deductive'
        },
        
        'universal_generalization': {
            'name': '全稱概括',
            'pattern': 'P(a) ⊢ ∀xP(x)',
            'validity': 0.95,  # 需要額外條件
            'category': 'inductive'
        },
        
        'existential_instantiation': {
            'name': '存在例化',
            'pattern': '∃xP(x) ⊢ P(a)',
            'validity': 0.95,
            'category': 'deductive'
        },
        
        'causal_inference': {
            'name': '因果推論',
            'pattern': 'C → E, do(C) ⊢ E',
            'validity': 0.80,
            'category': 'causal'
        },
        
        'abductive_inference': {
            'name': '溯因推論',
            'pattern': 'E, E ← P ⊢ P',
            'validity': 0.70,
            'category': 'abductive'
        }
    }
    
    def match(self, premise: Statement, conclusion: Statement) -> InferenceRule:
        """
        匹配適用的推論規則
        
        返回最匹配的規則，或 None
        """
        
        for rule_id, rule_def in self.RULES.items():
            if self._pattern_matches(rule_def['pattern'], premise, conclusion):
                return InferenceRule(
                    id=rule_id,
                    definition=rule_def,
                    confidence=rule_def['validity']
                )
        
        return None
    
    def _pattern_matches(self, pattern: str, premise: Statement, conclusion: Statement) -> bool:
        """
        檢查命題是否匹配推論模式
        """
        
        # 簡化的模式匹配實現
        # 實際實現需要命題邏輯 parser
        
        pattern_parts = pattern.split(' ⊢ ')
        if len(pattern_parts) != 2:
            return False
        
        premise_pattern = pattern_parts[0].strip()
        conclusion_pattern = pattern_parts[1].strip()
        
        # 結構匹配檢查
        return self._structural_match(premise, premise_pattern) and \
               self._structural_match(conclusion, conclusion_pattern)
```

---

## §2. 證明鏈建構

### §2.1 自動證明鏈建構

```python
class ProofChainConstructor:
    """
    證明鏈建構器
    
    根據目標結論自動建構證明鏈
    """
    
    def __init__(self):
        self.axiom_set = AxiomSet()
        self.lemma_registry = LemmaRegistry()
        self.causal_engine = CausalInferenceEngine()
        self.search_strategy = 'bidirectional'  # 雙向搜索
    
    def construct_proof(self, target: Proposition, premises: List[Proposition]) -> ProofChain:
        """
        建構從前提到目標結論的證明鏈
        
        策略：
        1. 正向搜索：從前提向結論推進
        2. 反向搜索：從結論向前提回溯
        3. 雙向搜索：結合前兩者
        """
        
        proof = ProofChain(target=target)
        
        if self.search_strategy == 'forward':
            proof = self._forward_search(target, premises)
        elif self.search_strategy == 'backward':
            proof = self._backward_search(target, premises)
        elif self.search_strategy == 'bidirectional':
            proof = self._bidirectional_search(target, premises)
        
        return proof
    
    def _bidirectional_search(self, target: Proposition, premises: List[Proposition]) -> ProofChain:
        """
        雙向搜索建構證明鏈
        
        從兩端同時搜索，在中間會合
        """
        
        # 前向 frontier：從前提出發可達的命題
        forward_frontier = set(premises)
        forward_visited = set(premises)
        forward_parents = {}  # 記錄前向路徑
        
        # 後向 frontier：可導出目標的命題
        backward_frontier = {target}
        backward_visited = {target}
        backward_parents = {}  # 記錄後向路徑
        
        max_iterations = 1000
        iteration = 0
        
        while forward_frontier and backward_frontier and iteration < max_iterations:
            iteration += 1
            
            # 選擇 frontier 較小的一端擴展
            if len(forward_frontier) <= len(backward_frontier):
                # 向前擴展一步
                new_frontier = set()
                
                for proposition in forward_frontier:
                    # 應用所有推論規則
                    for rule in InferenceRuleRegistry.RULES.values():
                        implied = rule.apply_forward(proposition)
                        
                        for imp in implied:
                            if imp not in forward_visited:
                                forward_visited.add(imp)
                                new_frontier.add(imp)
                                forward_parents[imp] = (proposition, rule.id)
                
                forward_frontier = new_frontier
                
                # 檢查是否與後向 frontier 相遇
                intersection = forward_frontier & backward_frontier
                if intersection:
                    meeting_point = intersection.pop()
                    return self._reconstruct_proof(
                        meeting_point,
                        forward_parents,
                        backward_parents,
                        premises,
                        target
                    )
            
            else:
                # 向後擴展一步
                new_frontier = set()
                
                for proposition in backward_frontier:
                    # 應用反向推論規則
                    for rule in InferenceRuleRegistry.RULES.values():
                        required = rule.apply_backward(proposition)
                        
                        for req in required:
                            if req not in backward_visited:
                                backward_visited.add(req)
                                new_frontier.add(req)
                                backward_parents[req] = (proposition, rule.id)
                
                backward_frontier = new_frontier
                
                # 檢查是否與前向 frontier 相遇
                intersection = forward_frontier & backward_frontier
                if intersection:
                    meeting_point = intersection.pop()
                    return self._reconstruct_proof(
                        meeting_point,
                        forward_parents,
                        backward_parents,
                        premises,
                        target
                    )
        
        # 無法建構完整證明
        return ProofChain(
            target=target,
            status='INCOMPLETE',
            reached_frontier=forward_frontier | backward_frontier
        )
    
    def _reconstruct_proof(self, meeting_point, forward_parents, backward_parents, 
                          premises, target) -> ProofChain:
        """
        重建完整證明鏈
        """
        
        nodes = []
        edges = []
        
        # 追蹤前向路徑
        current = meeting_point
        path_nodes = []
        
        while current not in premises:
            if current in forward_parents:
                parent, rule_id = forward_parents[current]
                path_nodes.append(current)
                current = parent
            else:
                break
        
        path_nodes.reverse()
        
        # 添加前向路徑
        for node in path_nodes:
            nodes.append(ProofNode(id=node.id, proposition=node))
        
        # 添加 meeting_point
        nodes.append(ProofNode(id=meeting_point.id, proposition=meeting_point))
        
        # 追蹤後向路徑
        current = meeting_point
        
        while current != target:
            if current in backward_parents:
                child, rule_id = backward_parents[current]
                edges.append(ProofEdge(
                    from_node=current,
                    to_node=child,
                    rule_id=rule_id
                ))
                current = child
                nodes.append(ProofNode(id=current.id, proposition=current))
            else:
                break
        
        return ProofChain(
            nodes=nodes,
            edges=edges,
            target=target,
            status='COMPLETE'
        )
```

### §2.2 證明壓縮

```python
class ProofCompressor:
    """
    證明壓縮器
    
    識別證明中的冗餘步驟，產生更緊湊的證明
    """
    
    def compress(self, proof: ProofChain) -> ProofChain:
        """
        壓縮證明鏈
        
        方法：
        1. 識別可直接跳過的中間步驟
        2. 合併等價推論
        3. 消除重複子證明
        """
        
        compressed = ProofChain(target=proof.target)
        
        # 步驟 1：去除冗餘節點
        essential_nodes = self._find_essential_nodes(proof)
        
        # 步驟 2：合併等價推論
        merged_edges = self._merge_equivalent_inferences(proof.edges)
        
        # 步驟 3：重構壓縮後的證明
        compressed.nodes = essential_nodes
        compressed.edges = merged_edges
        
        return compressed
    
    def _find_essential_nodes(self, proof: ProofChain) -> List[ProofNode]:
        """
        識別 essential nodes
        
        使用節點支配關係：
        一個節點是 essential 的，如果所有從起點到終點的路徑都經過它
        """
        
        essential = []
        
        # 構建圖結構
        graph = nx.DiGraph()
        for node in proof.nodes:
            graph.add_node(node.id)
        for edge in proof.edges:
            graph.add_edge(edge.from_node, edge.to_node)
        
        # 計算所有路徑
        source_nodes = [n for n in proof.nodes if not graph.predecessors(n.id)]
        target_nodes = [n for n in proof.nodes if not graph.successors(n.id)]
        
        for node in proof.nodes:
            if node.id in [n.id for n in source_nodes] or \
               node.id in [n.id for n in target_nodes]:
                essential.append(node)
                continue
            
            # 檢查是否所有路徑都經過此節點
            all_paths = list(nx.all_simple_paths(graph, 
                source_nodes[0].id, target_nodes[0].id))
            
            paths_through_node = [p for p in all_paths if node.id in p]
            
            if len(paths_through_node) == len(all_paths):
                essential.append(node)
        
        return essential
```

---

## §3. 公理/引理/定理追溯

### §3.1 源追溯引擎

```python
class SourceTracer:
    """
    源追溯引擎
    
    追溯每個證明步驟的來源
    """
    
    def __init__(self):
        self.axiom_registry = AxiomRegistry()
        self.lemma_registry = LemmaRegistry()
        self.theorem_registry = TheoremRegistry()
        self.causal_graph_store = CausalGraphStore()
    
    def trace_source(self, step: ProofStep) -> SourceTrace:
        """
        追溯推論步驟的來源
        
        返回完整的溯源鏈
        """
        
        trace = SourceTrace(step_id=step.id)
        
        if step.source_type == 'axiom':
            trace = self._trace_axiom(step)
        elif step.source_type == 'lemma':
            trace = self._trace_lemma(step)
        elif step.source_type == 'theorem':
            trace = self._trace_theorem(step)
        elif step.source_type == 'causal':
            trace = self._trace_causal(step)
        elif step.source_type == 'assumption':
            trace = self._trace_assumption(step)
        
        return trace
    
    def _trace_axiom(self, step: ProofStep) -> SourceTrace:
        """
        追溯至公理
        """
        
        axiom = self.axiom_registry.get(step.source_id)
        
        return SourceTrace(
            step_id=step.id,
            source_type='axiom',
            source_id=step.source_id,
            source_name=axiom.name,
            source_definition=axiom.definition,
            confidence=1.0,
            provenance_chain=[{
                'type': 'axiom',
                'id': axiom.id,
                'name': axiom.name,
                'immutable': axiom.is_immutable_core
            }]
        )
    
    def _trace_lemma(self, step: ProofStep) -> SourceTrace:
        """
        追溯至引理
        """
        
        lemma = self.lemma_registry.get(step.source_id)
        
        # 遞歸追溯引理的來源
        lemma_proof_trace = self._trace_proof(lemma.proof)
        
        return SourceTrace(
            step_id=step.id,
            source_type='lemma',
            source_id=step.source_id,
            source_name=lemma.name,
            source_definition=lemma.statement,
            confidence=0.95,
            provenance_chain=[{
                'type': 'lemma',
                'id': lemma.id,
                'name': lemma.name,
                'verification_status': lemma.verification_status
            }] + lemma_proof_trace.provenance_chain
        )
    
    def _trace_theorem(self, step: ProofStep) -> SourceTrace:
        """
        追溯至定理
        """
        
        theorem = self.theorem_registry.get(step.source_id)
        
        # 追溯定理的完整證明鏈
        theorem_proof_trace = self._trace_proof(theorem.proof)
        
        return SourceTrace(
            step_id=step.id,
            source_type='theorem',
            source_id=step.source_id,
            source_name=theorem.name,
            source_definition=theorem.statement,
            confidence=0.99,
            provenance_chain=[{
                'type': 'theorem',
                'id': theorem.id,
                'name': theorem.name,
                'proof_date': theorem.proof_date,
                'proof_authority': theorem.proof_authority
            }] + theorem_proof_trace.provenance_chain
        )
    
    def _trace_causal(self, step: ProofStep) -> SourceTrace:
        """
        追溯至因果圖
        """
        
        causal_graph = self.causal_graph_store.get(step.causal_graph_id)
        
        return SourceTrace(
            step_id=step.id,
            source_type='causal',
            source_id=step.causal_graph_id,
            source_name=causal_graph.name,
            confidence=0.80,
            provenance_chain=[{
                'type': 'causal_graph',
                'id': causal_graph.id,
                'nodes': causal_graph.node_count,
                'edges': causal_graph.edge_count,
                'causal_mechanism': causal_graph.mechanism
            }]
        )
    
    def _trace_proof(self, proof: ProofChain) -> SourceTrace:
        """
        遞歸追溯證明鏈
        """
        
        provenance_chain = []
        
        for step in proof.steps:
            step_trace = self.trace_source(step)
            provenance_chain.append({
                'step_id': step.id,
                'source_type': step.source_type,
                'source_id': step.source_id
            })
            
            # 遞歸處理
            if step.source_type in ['lemma', 'theorem']:
                sub_trace = self._trace_proof(step.sub_proof)
                provenance_chain.extend(sub_trace.provenance_chain)
        
        return SourceTrace(
            provenance_chain=provenance_chain
        )
```

### §3.2 溯源完整性評估

```python
class ProvenanceCompletenessEvaluator:
    """
    溯源完整性評估器
    
    評估證明鏈的溯源完整性
    """
    
    def evaluate(self, proof_chain: ProofChain) -> ProvenanceCompletenessResult:
        """
        評估溯源完整性
        """
        
        completeness_scores = []
        missing_sources = []
        incomplete_traces = []
        
        for step in proof_chain.steps:
            trace = SourceTracer().trace_source(step)
            
            # 計算每步的溯源分數
            score = self._calculate_completeness_score(trace)
            completeness_scores.append(score)
            
            if score < 1.0:
                incomplete_traces.append({
                    'step_id': step.id,
                    'score': score,
                    'trace': trace
                })
            
            # 檢查是否有缺失來源
            if step.source_type == 'unknown':
                missing_sources.append({
                    'step_id': step.id,
                    'conclusion': step.conclusion
                })
        
        # 計算整體完整性分數
        overall_score = sum(completeness_scores) / len(completeness_scores) \
                       if completeness_scores else 0.0
        
        # 識別溯源類型分佈
        source_distribution = self._calculate_distribution(proof_chain)
        
        return ProvenanceCompletenessResult(
            overall_score=overall_score,
            score_breakdown={
                'axiom_sourced': source_distribution.get('axiom', 0),
                'lemma_sourced': source_distribution.get('lemma', 0),
                'theorem_sourced': source_distribution.get('theorem', 0),
                'causal_sourced': source_distribution.get('causal', 0),
                'assumption_sourced': source_distribution.get('assumption', 0),
                'unknown_sourced': source_distribution.get('unknown', 0)
            },
            missing_sources=missing_sources,
            incomplete_traces=incomplete_traces,
            recommendation=self._generate_recommendation(overall_score)
        )
    
    def _calculate_completeness_score(self, trace: SourceTrace) -> float:
        """
        計算溯源完整性分數
        """
        
        if trace.source_type == 'axiom':
            return 1.0
        elif trace.source_type == 'theorem':
            return 0.99
        elif trace.source_type == 'lemma':
            return 0.95
        elif trace.source_type == 'causal':
            return 0.80
        elif trace.source_type == 'assumption':
            return 0.50
        else:
            return 0.0
    
    def _generate_recommendation(self, score: float) -> str:
        """
        根據完整性分數生成建議
        """
        
        if score >= 0.95:
            return 'Provenance is complete. Proof is highly trustworthy.'
        elif score >= 0.80:
            return 'Provenance is mostly complete. Consider adding more formal sources.'
        elif score >= 0.60:
            return 'Provenance is partial. Strengthen proof with verified lemmas.'
        elif score >= 0.40:
            return 'Provenance is weak. Require causal validation or external verification.'
        else:
            return 'Provenance is insufficient. Reject or require complete reconstruction.'
```

---

## §4. 審計與記錄

### §4.1 證明檢查日誌

```python
def LogProofCheckToAuditTrail(check_result, proof_chain):
    """
    將證明鏈檢查結果記錄至 AUDIT_TRAIL
    """
    
    audit_entry = {
        'event': 'PROOF_CHECK',
        'timestamp': GetCurrentTimestamp(),
        
        # 證明識別
        'proof_id': proof_chain.id,
        'target': str(proof_chain.target),
        
        # 檢查結果
        'structure_valid': check_result.structure_valid,
        'inference_valid': check_result.inference_valid,
        'source_traceable': check_result.source_traceable,
        
        # 溯源完整性
        'provenance_score': check_result.provenance_score,
        'axiom_count': check_result.axiom_count,
        'lemma_count': check_result.lemma_count,
        'theorem_count': check_result.theorem_count,
        'causal_count': check_result.causal_count,
        'assumption_count': check_result.assumption_count,
        
        # 問題列表
        'issues': check_result.issues,
        
        # 信心度
        'confidence': check_result.confidence
    }
    
    AppendToAuditTrail(audit_entry)
    
    return audit_entry
```

---

## §5. 版本與演化

| 版本 | 日期 | 變更摘要 |
|------|------|----------|
| v2.2 | 2026-03 | 初始版本，建立證明鏈檢查框架 |

---

*NoieLogicAGENTS — PROOF_CHECKER 模組*  
*Logic-OS v2.2 證明鏈檢查核心*  
*驗證有效性、建構完整性、追溯來源*
