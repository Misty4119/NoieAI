# NoieLogicAGENTS — 論理閉包検出器

|**バージョン：** Logic-OS v2.2  
|**モジュールコード：** CLOSURE_DETECTOR  
|**責務：** 論理閉包計算、缺失ステップ識別、閉包完全性評価  
|**親モジュール：** FORMAL_VERIFIER
|
---

|> ⚠️ 重要安全と意思決定プロトコル (CRITICAL SAFETY & DECISION PROTOCOL v2.2):
|> 1. CONSTRAINTS.md と社会権限レベル (SA-L0 〜 SA-L5) を厳守すること。
|> 2. 因果推論：すべての意思決定は因果グラフ（DAG）に基づき、因果メカニズムを标注すること。
|> 3. 主客分離：意思決定推論は自己状態と環境状態を混同してはならない。
|> 4. 形式検証：高リスク意思決定パスは論理閉包検証に合格する必要がある。
|> 5. シャドウシミュレーション：SA-L3+ 操作涉及時は、SANDBOX で結果を予行練習すること。
|> 6. 情報ビット完全性：情報ビットを捏造してはならない。KNOWLEDGE_BASE が空の場合は、「データ欠落」を明確に宣言すること。
|> 7. 認知リソース制約：意思決定の深さは利用可能な認知リソースを超えてはならない。
|> 8. 監査：すべての競合、拒否、形式検証結果を AUDIT_TRAIL に記録すること。
|> 9. 生存優先：すべての意思決定は実行前に吸収状態につながらないことを検証すること。
|> 10. 自己進化：公理系が進化する際、不変コアは保持されなければならない。
|
---

## §0. 論理閉包フレームワーク

### §0.1 コア定義

```text
【論理閉包の定義】

命題の集合 S が与えられた場合、論理的閉包 CL(S) は以下のように定義される：

  CL(S) = { φ | S ⊢ φ }

つまり：S から論理的に導出可能なすべての命題の集合。

形式的性質：

  1. 包含性：S ⊆ CL(S)
  2. 巾等性：CL(CL(S)) = CL(S)
  3. 単調性：S ⊆ T ならば CL(S) ⊆ CL(T)

【閉包演算子】

  Closure(P) = S₀ ∪ S₁ ∪ S₂ ∪ ...
  
  ただし：
    S₀ = P (命題集合)
    Sᵢ₊₁ = Sᵢ ∪ { φ | ∃r ∈ R, ∃s ∈ Sᵢ: r(s) = φ }
  
  Sᵢ₊₁ = Sᵢ（安定点）まで反復
```

### §0.2 閉包完全性

```text
【閉包完全性の定義】

証明チェーン P が閉包完全とは、当且つ当の場合：

  Closure(P) ⊆ P

つまり：P から論理的に導出可能なすべての結論が P の中ですでに明確に陈述されている。

【缺失ステップ】

缺失ステップとは：
  φ ∈ Closure(P) かつ φ ∉ P

これらは論理的にな必要だが明確に陈述されていない推論ステップである。
```

---

## §1. 論理的閉包計算

### §1.1 閉包計算エンジン

```python
class LogicalClosureEngine:
    """
    論理的閉包計算エンジン
    
    命題集合の論理的閉包を計算
    """
    
    def __init__(self):
        self.inference_rules = InferenceRuleRegistry()
        self.axiom_set = AxiomSet()
        self.max_iterations = 1000  # 無限ループを防止
        self.closure_cache = {}
    
    def compute_closure(self, proposition_set: Set[Proposition]) -> Set[Proposition]:
        """
        命題集合の閉包を計算
        
        方法：推論規則を安定点まで反復的に適用
        
        返値：
            閉包集合 (Closure Set)
        """
        
        # キャッシュを検査
        cache_key = self._hash_proposition_set(proposition_set)
        if cache_key in self.closure_cache:
            return self.closure_cache[cache_key]
        
        # 初期化
        current_closure = set(proposition_set)
        previous_closure = set()
        
        iteration = 0
        
        while current_closure != previous_closure and iteration < self.max_iterations:
            iteration += 1
            previous_closure = set(current_closure)
            
            # すべての推論規則を適用
            new_propositions = self._apply_inference_rules(current_closure)
            
            # 新規命題を追加
            current_closure.update(new_propositions)
        
        if iteration >= self.max_iterations:
            # 警告を記録：無限ループの可能性がある
            self._log_iteration_warning(proposition_set, iteration)
        
        # 結果をキャッシュ
        self.closure_cache[cache_key] = current_closure
        
        return current_closure
    
    def _apply_inference_rules(self, propositions: Set[Proposition]) -> Set[Proposition]:
        """
        推論規則を適用して新規命題を生成
        """
        
        new_propositions = set()
        
        # すべての命題ペアを走査
        proposition_list = list(propositions)
        
        for i, prop_i in enumerate(proposition_list):
            # 単命題推論（単項規則）
            unary_results = self._apply_unary_rules(prop_i)
            new_propositions.update(unary_results)
            
            # 命題ペア推論（二項規則）
            for prop_j in proposition_list[i+1:]:
                binary_results = self._apply_binary_rules(prop_i, prop_j)
                new_propositions.update(binary_results)
        
        # すでに存在する命題をフィルター
        new_propositions = new_propositions - propositions
        
        return new_propositions
    
    def _apply_unary_rules(self, proposition: Proposition) -> Set[Proposition]:
        """
        単項推論規則を適用
        
        例：
        - 否定消除：¬¬P → P
        - 二重否定導入：P → ¬¬P
        """
        
        results = set()
        
        # 規則 1：否定消除
        if proposition.is_double_negation:
            results.add(proposition.eliminate_double_negation())
        
        # 規則 2：合取消除
        if proposition.is_conjunction:
            results.add(proposition.conjunct_1)
            results.add(proposition.conjunct_2)
        
        # 規則 3：条件伝播
        if proposition.is_conditional:
            # P → Q から ¬Q → ¬P (対偶) を導出
            results.add(Proposition.contrapositive(proposition))
        
        return results
    
    def _apply_binary_rules(self, prop_a: Proposition, prop_b: Proposition) -> Set[Proposition]:
        """
        二項推論規則を適用
        
        例：
        - 肯定前件：(P → Q), P ⊢ Q
        - 選言三段論：(P ∨ Q), ¬P ⊢ Q
        - 仮言三段論：(P → Q), (Q → R) ⊢ (P → R)
        """
        
        results = set()
        
        # 規則：Modus Ponens (肯定前件)
        if prop_a.is_conditional and prop_b.entails(prop_a.antecedent):
            results.add(prop_a.consequent)
        
        # 規則：Modus Tollens (否定後件)
        if prop_a.is_conditional and prop_b.entails(Proposition.negate(prop_a.consequent)):
            results.add(Proposition.negate(prop_a.antecedent))
        
        # 規則：仮言三段論 (Hypothetical Syllogism)
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
        
        # 規則：選言導入 (Disjunction Introduction)
        results.add(Proposition.disjunction(prop_a, prop_b))
        results.add(Proposition.disjunction(prop_b, prop_a))
        
        return results
```

### §1.2 増分閉包計算

```python
class IncrementalClosureEngine:
    """
    増分閉包計算エンジン
    
    命題集合が変化した時、閉包を増分更新
    """
    
    def __init__(self):
        self.base_closure = None
        self.base_set = None
        self.incremental_engine = LogicalClosureEngine()
    
    def compute_incremental(self, original_set: Set[Proposition], 
                           added: Set[Proposition],
                           removed: Set[Proposition]) -> Set[Proposition]:
        """
        増分計算で閉包を計算
        
        戦略：
        1. 追加のみの場合：追加命題の閉包を計算し、元閉包とマージ
        2. 削除のみの場合：再計算が必要（閉包は削除をサポートしない）
        3. 両方ある場合：まず追加を処理してから再計算
        """
        
        if removed and not added:
            # 完全再計算が必要
            new_set = original_set - removed
            return self.incremental_engine.compute_closure(new_set)
        
        if added and not removed:
            # 増分更新
            # 戦略：追加命題の元閉包に対する増分閉包を計算
            
            if self.base_closure is None:
                self.base_closure = self.incremental_engine.compute_closure(original_set)
                self.base_set = original_set
            
            # 追加命題の元閉包に対する増分閉包
            combined = self.base_closure | added
            incremental_closure = self.incremental_engine.compute_closure(combined)
            
            # 増分 = 新閉包 - 元閉包（追加命題を含む）
            incremental = incremental_closure - self.base_closure
            
            return self.base_closure | incremental
        
        # 両方ある場合
        new_set = (original_set | added) - removed
        return self.incremental_engine.compute_closure(new_set)
    
    def invalidate_cache(self):
        """
        キャッシュを無効化
        
        推論規則または公理が変化した時呼び出す
        """
        
        self.base_closure = None
        self.base_set = None
```

---

## §2. 缺失ステップ識別

### §2.1 缺失ステップ検出器

```python
class MissingStepDetector:
    """
    缺失ステップ検出器
    
    証明チェーンで缺失の論理的ステップを識別
    """
    
    def __init__(self):
        self.closure_engine = LogicalClosureEngine()
    
    def detect_missing_steps(self, proof_chain: ProofChain) -> MissingStepResult:
        """
        缺失ステップを検出
        
        ステップ：
        1. 証明チェーン命題集合の閉包を計算
        2. 閉包にあるが元の証明にない命題を識別
        3. 各缺失命題について、必要な推論の識別を試みる
        """
        
        # ステップ 1：命題集合を抽出
        proposition_set = set(step.conclusion for step in proof_chain.steps)
        
        # ステップ 2：閉包を計算
        closure = self.closure_engine.compute_closure(proposition_set)
        
        # ステップ 3：缺失命題を識別
        missing_propositions = closure - proposition_set
        
        # ステップ 4：各缺失命題について分析
        missing_steps = []
        
        for missing_prop in missing_propositions:
            # 既存命題から缺失命題への推論パスを探す
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
                # 自動識別不可能、人工介入が必要
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
        前提から目標命題への推論パスを探す
        """
        
        # 双方向検索を使用
        # 前方向：前提から目標へ検索
        # 後方向：目標から前提へ検索
        
        forward_frontier = set(premises)
        forward_visited = set(premises)
        forward_paths = {p: [p] for p in premises}
        
        backward_frontier = {target}
        backward_visited = {target}
        backward_paths = {target: [target]}
        
        max_depth = 10
        
        for depth in range(max_depth):
            # 前方向に拡張
            new_forward = set()
            for prop in forward_frontier:
                # 前方向推論規則を適用
                results = self._apply_forward_rules(prop)
                for result in results:
                    if result not in forward_visited:
                        forward_visited.add(result)
                        new_forward.add(result)
                        forward_paths[result] = forward_paths[prop] + [result]
            
            # 目標を発見したか检查
            if target in new_forward:
                return InferencePath(
                    premises=premises,
                    target=target,
                    path=forward_paths[target],
                    rule='composite',
                    confidence=0.8 ** depth
                )
            
            forward_frontier = new_forward
            
            # 後方向に拡張
            new_backward = set()
            for prop in backward_frontier:
                # 逆方向推論規則を適用
                requirements = self._apply_backward_rules(prop)
                for req in requirements:
                    if req not in backward_visited:
                        backward_visited.add(req)
                        new_backward.add(req)
                        backward_paths[req] = [req] + backward_paths[prop]
            
            backward_frontier = new_backward
            
            # 合流点があるか检查
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
        前方向推論規則を適用
        """
        
        results = set()
        
        # 合取消除
        if proposition.is_conjunction:
            results.add(proposition.conjunct_1)
            results.add(proposition.conjunct_2)
        
        # 条件前件
        # より複雑な実装が必要
        
        return results
    
    def _apply_backward_rules(self, proposition: Proposition) -> Set[Proposition]:
        """
        逆方向推論規則を適用
        """
        
        results = set()
        
        # 合取引入：二つの前提が必要
        if proposition.is_conjunction:
            results.add(proposition.conjunct_1)
            results.add(proposition.conjunct_2)
        
        # 選言消除：分岐議論が必要
        if proposition.is_disjunction:
            # 空集合を返す更多信息が必要
            pass
        
        return results
    
    def _calculate_severity(self, inference_path: InferencePath) -> str:
        """
        缺失ステップの重大性を計算
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

### §2.2 自動補完

```python
class ProofCompleter:
    """
    証明自動補完器
    
    缺失の推論ステップを自動補足
    """
    
    def __init__(self):
        self.missing_step_detector = MissingStepDetector()
        self.inference_engine = LogicalClosureEngine()
    
    def complete_proof(self, proof_chain: ProofChain) -> ProofChain:
        """
        証明チェーンを自動補完
        """
        
        # 缺失ステップを検出
        missing_result = self.missing_step_detector.detect_missing_steps(proof_chain)
        
        # 元証明をコピー
        completed = ProofChain(
            id=proof_chain.id,
            target=proof_chain.target,
            nodes=list(proof_chain.nodes),
            edges=list(proof_chain.edges)
        )
        
        # 非致命的缺失ステップを追加
        auto_completed_count = 0
        
        for missing in missing_result.missing_steps:
            if missing.severity in ['LOW', 'MEDIUM'] and not missing.requires_manual:
                # 自動補足
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
        
        # 補完結果を記録
        self._log_completion(proof_chain.id, auto_completed_count, 
                           missing_result.total_missing)
        
        return completed
    
    def _log_completion(self, proof_id, completed_count, total_missing):
        """
        補完ログを記録
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

## §3. 閉包完全性評価

### §3.1 完全性指標

```python
class ClosureCompletenessEvaluator:
    """
    閉包完全性評価器
    
    証明チェーンの閉包完全性を評価
    """
    
    def evaluate(self, proof_chain: ProofChain) -> ClosureCompletenessResult:
        """
        閉包完全性を評価
        """
        
        # 命題集合を計算
        proposition_set = set(step.conclusion for step in proof_chain.steps)
        
        # 閉包を計算
        closure_engine = LogicalClosureEngine()
        closure = closure_engine.compute_closure(proposition_set)
        
        # 指標を計算
        missing = closure - proposition_set
        
        # 完全性比率
        completeness_ratio = len(proposition_set) / len(closure) if closure else 0.0
        
        # 閉包密度
        closure_density = len(closure) / (len(proposition_set) ** 2) if proposition_set else 0.0
        
        # 閉包内のタイプ分布を識別
        type_distribution = self._analyze_closure_types(closure, proposition_set)
        
        return ClosureCompletenessResult(
            proof_id=proof_chain.id,
            
            # 基本指標
            proposition_count=len(proposition_set),
            closure_count=len(closure),
            missing_count=len(missing),
            
            # 完全性評価
            completeness_ratio=completeness_ratio,
            completeness_score=self._score_completeness(completeness_ratio),
            
            # 構造指標
            closure_density=closure_density,
            
            # タイプ分布
            explicit_ratio=type_distribution['explicit'],
            implicit_ratio=type_distribution['implicit'],
            
            # 缺失分析
            missing_propositions=list(missing),
            missing_by_severity=self._categorize_by_severity(proof_chain, missing),
            
            # 提案
            recommendations=self._generate_recommendations(completeness_ratio, missing)
        )
    
    def _score_completeness(self, ratio: float) -> float:
        """
        完全性比率に基づいて評価
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
        閉包内の命題タイプを分析
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
        重大性によって缺失命題を分類
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
        改善提案を生成
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

### §3.2 完全性レポート生成

```python
def generate_closure_completeness_report(proof_chain: ProofChain) -> str:
    """
    閉包完全性レポートを生成
    """
    
    evaluator = ClosureCompletenessEvaluator()
    result = evaluator.evaluate(proof_chain)
    
    report = f"""
================================================================================
                    論理的閉包完全性レポート
================================================================================

証明 ID: {result.proof_id}

--------------------------------------------------------------------------------
基本指標
--------------------------------------------------------------------------------
命題数:    {result.proposition_count}
閉包サイズ:    {result.closure_count}
缺失数:    {result.missing_count}

--------------------------------------------------------------------------------
完全性評価
--------------------------------------------------------------------------------
完全性比率:  {result.completeness_ratio:.2%}
完全性スコア:  {result.completeness_score:.2f}

構造指標:
- 閉包密度: {result.closure_density:.4f}

タイプ分布:
- 明確な命題: {result.explicit_ratio:.2%}
- 暗黙の命題: {result.implicit_ratio:.2%}

--------------------------------------------------------------------------------
缺失分析
--------------------------------------------------------------------------------
"""
    
    for severity, steps in result.missing_by_severity.items():
        if steps:
            report += f"\n{severity} 重大性 ({len(steps)} 項):\n"
            for step in steps[:5]:  # 最大5項目を表示
                report += f"  - {step.proposition}\n"
            if len(steps) > 5:
                report += f"  ... 他 {len(steps) - 5} 項\n"
    
    report += """
--------------------------------------------------------------------------------
提案
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

## §4. 監査と記録

### §4.1 閉包検出ログ

```python
def LogClosureDetectionToAuditTrail(detection_result, proof_chain):
    """
    閉包検出結果を AUDIT_TRAIL に記録
    """
    
    audit_entry = {
        'event': 'CLOSURE_DETECTION',
        'timestamp': GetCurrentTimestamp(),
        
        # 証明識別
        'proof_id': proof_chain.id,
        
        # 検出結果
        'completeness_ratio': detection_result.completeness_ratio,
        'completeness_score': detection_result.completeness_score,
        
        # 数量指標
        'proposition_count': detection_result.proposition_count,
        'closure_count': detection_result.closure_count,
        'missing_count': detection_result.missing_count,
        
        # 缺失分類
        'missing_low': len(detection_result.missing_by_severity.get('LOW', [])),
        'missing_medium': len(detection_result.missing_by_severity.get('MEDIUM', [])),
        'missing_high': len(detection_result.missing_by_severity.get('HIGH', [])),
        'missing_critical': len(detection_result.missing_by_severity.get('CRITICAL', [])),
        
        # 構造指標
        'closure_density': detection_result.closure_density,
        
        # 提案
        'recommendations': detection_result.recommendations
    }
    
    AppendToAuditTrail(audit_entry)
    
    return audit_entry
```

---

## §5. バージョンと進化

|| バージョン | 日付 | 変更摘要 |
||------|------|----------|
|| v2.2 | 2026-03 | 初期バージョン、論理的閉包検出フレームワークを確立 |

---

*NoieLogicAGENTS — CLOSURE_DETECTOR モジュール*  
*Logic-OS v2.2 論理的閉包検出コア*  
*閉包の計算、缺失の識別、完全性評価*
