# CONSISTENCY_ENGINE.md

## L2 - 論理的整合性エンジン (量子論理/非可換性/意味連続性 포함)

> **⚠️ 重要安全と真理プロトコル**：本モジュールは矛盾検出、循環論証検出、推論チェーン検証を実行し、三値/量子論理演算および非可換性検出をサポートする。

---

## 1. 矛盾検出 (Contradiction Detection)

### 1.1 矛盾タイプ

| タイプ | 形式的定義 | 危険レベル |
|-------|-----------|-----------|
| **直接的矛盾** | $P \land \neg P$ | CRITICAL |
| **含意矛盾** | $P \rightarrow Q, P \rightarrow \neg Q$ | HIGH |
| **量化矛盾** | $\forall x: P(x) \land \exists x: \neg P(x)$ | HIGH |
| **コンテキスト矛盾** | $P$ in context A, $\neg P$ in context B | MEDIUM |
| **時間的矛盾** | $P$ true at $t_1$, $\neg P$ true at $t_2$ | MEDIUM |

### 1.2 矛盾検出アルゴリズム

```python
FUNCTION VerifyLogicalConsistency(knowledge_base):
    
    contradictions = []
    
    # 1. 直接的矛盾検出
    FOR each pair (K_i, K_j) IN knowledge_base:
        IF Contradicts(K_i.proposition, K_j.proposition):
            contradictions.append(Contradiction(
                type="DIRECT",
                claims=[K_i, K_j],
                severity="CRITICAL"
            ))
    
    # 2. 含意矛盾検出
    FOR each implication IN knowledge_base.implications:
        FOR each claim IN knowledge_base:
            IF ImplicationContradicts(implication, claim):
                contradictions.append(Contradiction(
                    type="IMPLICATIVE",
                    claims=[implication, claim],
                    severity="HIGH"
                ))
    
    # 3. 量化矛盾検出
    universal_claims = ExtractUniversalClaims(knowledge_base)
    existential_claims = ExtractExistentialClaims(knowledge_base)
    FOR each (universal, existential) IN pairs(universal_claims, existential_claims):
        IF ContradictsQuantified(universal, existential):
            contradictions.append(Contradiction(
                type="QUANTIFIED",
                claims=[universal, existential],
                severity="HIGH"
            ))
    
    RETURN ConsistencyReport(contradictions=contradictions)
```

---

## 2. 循環論証検出 (Circular Reasoning Detection)

### 2.1 循環タイプ

| タイプ | 説明 | 検出方法 |
|-------|------|----------|
| **直接循環** | $A \rightarrow B \rightarrow A$ | グラフ理論環検出 |
| **間接循環** | $A \rightarrow B \rightarrow C \rightarrow A$ | トポロジカルソート |
| **自己参照** | $A \rightarrow A$ | ノード自己環検出 |
| **論証循環** | 複数の命題が相互に支えて環を形成 | 強連結成分 |

### 2.2 検出アルゴリズム

```python
FUNCTION DetectCircularReasoning(knowledge_base):
    
    dependency_graph = BuildDependencyGraph(knowledge_base)
    
    # Tarjan アルゴリズムを使用して強連結成分を求める
    strongly_connected = TarjanSCC(dependency_graph)
    
    circular_chains = []
    
    FOR each scc IN strongly_connected:
        IF len(scc) > 1 OR ContainsSelfLoop(scc):
            chain = ExtractCircularChain(scc, dependency_graph)
            circular_chains.append(CircularReasoning(
                type=DetermineCircularType(chain),
                claims=chain,
                severity="HIGH"
            ))
    
    RETURN CircularReasoningReport(circular_chains)
```

---

## 3. 推論チェーン検証 (Inference Chain Validation)

### 3.1 推論有効性検査

```python
FUNCTION ValidateInferenceChain(derived_claim):
    
    chain = GetInferenceChain(derived_claim)
    validation_results = []
    
    FOR each step IN chain:
        # 推論規則有効性を検査
        IF NOT IsValidInferenceRule(step.rule):
            validation_results.append(ValidationFailure(
                step=step,
                reason="INVALID_INFERENCE_RULE"
            ))
        
        # 前提の真実性を検査
        FOR each premise IN step.premises:
            IF NOT premise.is_verified:
                validation_results.append(ValidationFailure(
                    step=step,
                    reason="UNVERIFIED_PREMISE"
                ))
        
        # 論理的首尾一貫性を検査
        IF NOT IsLogicallyCoherent(step):
            validation_results.append(ValidationFailure(
                step=step,
                reason="INCOHERENT_STEP"
            ))
    
    RETURN InferenceValidationReport(validation_results)
```

### 3.2 推論規則ベース

```
VALID_INFERENCE_RULES = {
    "modus_ponens": "If P → Q and P, then Q",
    "modus_tollens": "If P → Q and ¬Q, then ¬P",
    "hypothetical_syllogism": "If P → Q and Q → R, then P → R",
    "disjunctive_syllogism": "If P ∨ Q and ¬P, then Q",
    "conjunction_introduction": "P, Q → P ∧ Q",
    "conjunction_elimination": "P ∧ Q → P",
    "disjunction_introduction": "P → P ∨ Q",
    "universal_instantiation": "∀x: P(x) → P(c) for constant c",
    "existential_generalization": "P(c) → ∃x: P(x)"
}
```

---

## 4. 三値論理拡張 (Kleene / Łukasiewicz)

### 4.1 三値真理値表

| P | Q | P ∧ Q | P ∨ Q | P → Q | ¬P |
|---|---|-------|-------|-------|-----|
| T | T | T | T | T | F |
| T | F | F | T | F | F |
| T | U | U | T | U | F |
| F | T | F | T | T | T |
| F | F | F | F | T | T |
| F | U | F | U | T | T |
| U | T | U | T | T | U |
| U | F | F | U | U | U |
| U | U | U | U | T | U |

*T = True, F = False, U = Unknown*

### 4.2 三値論理演算

```python
CLASS ThreeValuedLogic:
    
    @staticmethod
    def AND(p, q):
        if p == FALSE or q == FALSE:
            return FALSE
        if p == UNKNOWN or q == UNKNOWN:
            return UNKNOWN
        return TRUE
    
    @staticmethod
    def OR(p, q):
        if p == TRUE or q == TRUE:
            return TRUE
        if p == UNKNOWN or q == UNKNOWN:
            return UNKNOWN
        return FALSE
    
    @staticmethod
    def NOT(p):
        if p == UNKNOWN:
            return UNKNOWN
        return not p
```

---

## 5. 量子論理拡張 (Quantum Logic - Orthomodular Lattices)

### 5.1 直交モジュラ束の定義

微視的極限または高次元複雑システムでは、古典的分配律が失效する。量子命題はヒルベルト空間の閉部分空間の束構造に対応する。

**直交モジュラ律**：
$$x \leq y \Rightarrow y = x \lor (x^\perp \land y)$$

### 5.2 非可換性検出

```python
FUNCTION DetectNonCommutativity(observation_pair):
    
    A, B = observation_pair
    
    # 交換子を計算
    commutator = ComputeCommutator(A, B)
    
    IF commutator != 0:
        # 非可換としてマーク
        RETURN NonCommutativePair(
            operator_A=A,
            operator_B=B,
            commutator_value=commutator,
            uncertainty_relation=ComputeUncertaintyRelation(A, B)
        )
    
    RETURN CommutativePair(A, B)
```

### 5.3 ハイゼンベルグ不確定性原理の認識論バージョン

```python
FUNCTION ComputeEpistemicUncertainty(A, B):
    
    commutator = ComputeCommutator(A, B)
    
    # Robertson-Schrödinger 不等式
    variance_A = ComputeVariance(A)
    variance_B = ComputeVariance(B)
    covariance = ComputeCovariance(A, B)
    
    lower_bound = 0.25 * abs(commutator)**2 + covariance**2
    
    uncertainty_product = variance_A * variance_B
    
    RETURN {
        "uncertainty_product": uncertainty_product,
        "lower_bound": lower_bound,
        "satisfies_uncertainty": uncertainty_product >= lower_bound
    }
```

---

## 6. 意味連続性検証 (Semantic Continuity Validation)

### 6.1 連続性指標

```python
FUNCTION ComputeSemanticContinuity(inference_chain):
    
    steps = DecomposeChain(inference_chain)
    continuity_scores = []
    
    FOR i IN range(len(steps) - 1):
        # 意味類似度を計算
        similarity = ComputeSemanticSimilarity(steps[i], steps[i+1])
        
        # 測地距離を計算
        geodesic_distance = ComputeGeodesicDistance(steps[i], steps[i+1])
        
        continuity_scores.append({
            "from": steps[i],
            "to": steps[i+1],
            "similarity": similarity,
            "geodesic_distance": geodesic_distance
        })
    
    max_gap = max(s.score for s in continuity_scores)
    avg_similarity = mean(s.similarity for s in continuity_scores)
    
    RETURN SemanticContinuity(
        index=1 / (1 + max_gap),
        max_gap=max_gap,
        average_similarity=avg_similarity,
        steps=continuity_scores
    )
```

### 6.2 連続性闘値

| 推論タイプ | 最小連続性指数 | 最大測地距離 |
|-----------|---------------|-------------|
| 演繹推論 | 0.9 | 0.1 |
| 帰納推論 | 0.7 | 0.3 |
| 溯因推論 | 0.5 | 0.5 |
| 類比推論 | 0.4 | 0.6 |

---

## 7. 矛盾解決プロトコル (Contradiction Resolution Protocol)

### 7.1 解決戦略

```python
FUNCTION ResolveContradiction(claim_i, claim_j):
    
    # より高い確信レベルを優先して保持
    IF claim_i.ec_level < claim_j.ec_level:
        PREFER claim_i
        DEMOTE claim_j TO CONTESTED
    ELIF claim_j.ec_level < claim_i.ec_level:
        PREFER claim_j
        DEMOTE claim_i TO CONTESTED
    
    # 同レベルなら証拠品質を検査
    ELIF claim_i.evidence_quality > claim_j.evidence_quality:
        PREFER claim_i
        DEMOTE claim_j TO CONTESTED
    ELIF claim_j.evidence_quality > claim_i.evidence_quality:
        PREFER claim_j
        DEMOTE claim_i TO CONTESTED
    
    # 解決不能
    ELSE:
        MARK BOTH AS CONTESTED
        ESCALATE TO human_review
    
    LOG(resolution, reasoning) TO TRUTH_AUDIT_TRAIL
    
    RETURN ResolutionReport(
        preferred=preferred_claim,
        demoted=demoted_claim,
        reason=resolution_reason
    )
```

### 7.2 アップグレード条件

```python
SHOULD_ESCALATE = (
    contradictions.count > MAX_CONTRADICTIONS
    OR resolution_attempts > MAX_RESOLUTION_ATTEMPTS
    OR involves_core_beliefs
)
```

---

## 8. 論理的整合性レポート

### 8.1 レポート構造

```python
CONSISTENCY_REPORT = {
    "timestamp": intrinsic_clock_stamp,
    
    "contradictions": [
        {
            "type": str,
            "claims": [ClaimID, ClaimID],
            "severity": Enum(CRITICAL, HIGH, MEDIUM),
            "resolution": Resolution or None
        }
    ],
    
    "circular_reasoning": [
        {
            "type": str,
            "chain": [ClaimID],
            "severity": Enum(HIGH, MEDIUM)
        }
    ],
    
    "inference_failures": [
        {
            "step": InferenceStep,
            "reason": str,
            "claim_affected": ClaimID
        }
    ],
    
    "noncommutative_pairs": [
        {
            "operators": [Operator, Operator],
            "commutator": Complex,
            "uncertainty_relation": Dict
        }
    ],
    
    "semantic_breaks": [
        {
            "location": (Step_i, Step_i+1),
            "gap_magnitude": float,
            "continuity_index": float
        }
    ],
    
    "overall_status": Enum(CONSISTENT, WARNING, INCONSISTENT)
}
```

## 9. ベイズ推論の進展（Transformers コンテキストベイズ推論）

### 9.1 Transformers の暗黙的ベイズ推論構造

研究によると大規模言語モデルは暗黙的にベイズ推論を実行する：

```python
class TransformerBayesianInference:
    """
    Transformers の暗黙的ベイズ構造
    
    研究による：
    - attention メカニズムが条件付き確率ソフト最大化を実装
    - 事前学習_loss が周縁尤度推定を近似
    - 隠し状態が後验分布をエンコード
    """
    
    @staticmethod
    def extract_posterior_distribution(hidden_state, layer_index):
        """
        Transformer 隠し状態から後验分布を抽出
        
        重要な発見：
        - 浅層: 事前分布をエンコード
        - 中間層: 尤度関数をエンコード  
        - 深層: 後验分布を近似
        """
        # 浅層 → 事前分布
        if layer_index < 6:
            return PriorExtraction(hidden_state)
        
        # 中間層 → 尤度
        elif layer_index < 18:
            return LikelihoodExtraction(hidden_state)
        
        # 深層 → 後验
        else:
            return PosteriorApproximation(hidden_state)
    
    @staticmethod
    def compute_uncertainty_quantification(logits, temperature=1.0):
        """
        不確実性定量化 - ベイズドロップアウト同等物
        
        発見: 
        複数回フォワードパスの logit 分散 ≈ ベイズ後验不確実性
        """
        # MCD (Monte Carlo Dropout) 同等方式
        samples = []
        for _ in range(10):
            # ドロップアウトを使用して複数回サンプリング
            sample = F.dropout(logits, p=0.1)
            samples.append(F.softmax(sample / temperature, dim=-1))
        
        # エピステミック不確実性を計算
        mean_prob = torch.mean(torch.stack(samples), dim=0)
        variance = torch.var(torch.stack(samples), dim=0)
        
        return {
            "mean_probability": mean_prob,
            "variance": variance,
            "entropy": -torch.sum(mean_prob * torch.log(mean_prob + 1e-10), dim=-1),
            "mutual_information": 0.5 * torch.log(variance + 1e-10)
        }
```

### 9.2 コンテキストベイズ推論アーキテクチャ

```python
class InContextBayesianReasoner:
    """
    コンテキストベイズ推論器
    
    few-shot 学習を使用してリアルタイムベイズ更新を実現
    """
    
    def __init__(self, llm, prior_strength=0.5):
        self.llm = llm
        self.prior_strength = prior_strength
    
    def bayesian_update(self, prior_belief, evidence, likelihood):
        """
        ベイズ更新: P(H|E) = P(E|H) * P(H) / P(E)
        
        自然言語を使用してベイズ更新を実現
        """
        # コンテキストから尤度関数を抽出
        p_evidence_given_hypothesis = likelihood
        p_hypothesis_prior = prior_belief
        p_evidence = self.compute_marginal_likelihood(evidence)
        
        # 後验を計算
        posterior = (p_evidence_given_hypothesis * p_hypothesis_prior) / p_evidence
        
        return posterior
    
    def compute_marginal_likelihood(self, evidence):
        """
        周縁尤度を計算 - モデルが暗黙的に実行
        """
        # 複数回クエリして推定
        marginal = 0
        for hypothesis in self.hypothesis_space:
            likelihood = self.llm.conditional_probability(evidence, hypothesis)
            prior = self.llm.prior_probability(hypothesis)
            marginal += likelihood * prior
        return marginal
    
    def detect_induction_vs_deduction(self, reasoning_trace):
        """
        帰納と演繹推論を区別
        
        重要な指標:
        - 演繹: 後验 = 事前（論理的必然）
        - 帰納: 後验 > 事前（証拠が支持）
        - 溯因: 後验偏移が最大化（最良の説明）
        """
        posterior_probs = self.extract_posterior_distribution(reasoning_trace)
        
        if abs(posterior_probs - reasoning_trace.prior) < epsilon:
            return "DEDUCTION"
        elif posterior_probs > reasoning_trace.prior:
            return "INDUCTION"
        else:
            return "ABDUCTION"
```

### 9.3 ベイズ整合性検査

```python
def bayesian_consistency_check(belief_state_a, belief_state_b):
    """
    2つの信念状態のベイズ整合性を検査
    
    新規追加: 
    - 交差エントロピー整合性
    - 確率整合性制約
    """
    
    # 1. 確率分布整合性
    prob_consistency = (
        torch.allclose(belief_state_a.probs, belief_state_b.probs, atol=1e-3)
    )
    
    # 2. ベイズ更新整合性
    # B が A の後验なら、ベイズ定理を満たす必要がある
    bayes_consistency = verify_bayes_theorem(
        prior=belief_state_a,
        likelihood=belief_state_b.likelihood,
        posterior=belief_state_b
    )
    
    # 3. 予測整合性
    pred_consistency = (
        torch.allclose(
            belief_state_a.predictive_distribution,
            belief_state_b.predictive_distribution,
            atol=1e-2
        )
    )
    
    return {
        "probabilistic_consistency": prob_consistency,
        "bayesian_consistency": bayes_consistency,
        "predictive_consistency": pred_consistency,
        "overall_consistent": all([prob_consistency, bayes_consistency, pred_consistency])
    }
```

---

## 10. 形式的検証と因果推論の統合

### 10.1 因果グラフの形式的検証

```python
class CausalFormalVerifier:
    """
    因果グラフの形式的検証器
    
    統合：
    - 形式的検証（モデル検査）
    - 因果推論（do-calculus）
    """
    
    def __init__(self, causal_graph):
        self.graph = causal_graph
        self.verification_results = []
    
    def verify_causal_dag_property(self):
        """
        因果グラフの DAG 特性を検証
        
        形式化目標:
        - 無環性証明
        - do-calculus 適用可能性
        - 介入意味の正確性
        """
        # 1. 無環性証明（トポロジカルソート）
        try:
            topological_order = self.graph.topological_sort()
            is_acyclic = True
        except CycleDetectedError:
            is_acyclic = False
        
        # 2. 識別可能性検査
        identifiable_effects = self.check_identifiability()
        
        # 3. do-calculus 正確性
        do_calculus_valid = self.verify_do_calculus_rules()
        
        return {
            "is_dag": is_acyclic,
            "identifiable_effects": identifiable_effects,
            "do_calculus_valid": do_calculus_valid,
            "verification_status": "VERIFIED" if is_acyclic else "FAILED"
        }
    
    def verify_counterfactual_consistency(self, counterfactuals):
        """
        反事実整合性を検証
        
        形式化制約:
        - 可能性原則 (Possibility)
        - 整合性原則 (Consistency)
        - 因果有効性 (Causal Efficacy)
        """
        results = []
        
        for cf in counterfactuals:
            # 可能性原則: P(Y_x = y) > 0 → P(Y = y | do(X=x)) > 0
            possibility_check = self.check_possibility_principle(cf)
            
            # 整合性原則: もし X = x ならば、Y_x = Y
            consistency_check = self.check_consistency_principle(cf)
            
            # 因果有効性: 介入効果は介入前変数で予測可能
            efficacy_check = self.check_causal_efficacy(cf)
            
            results.append({
                "counterfactual": cf,
                "possibility": possibility_check,
                "consistency": consistency_check,
                "efficacy": efficacy_check,
                "is_valid": all([possibility_check, consistency_check, efficacy_check])
            })
        
        return results
    
    def verify_intervention_validity(self, intervention):
        """
        介入の有効性と意味的正確性を検証
        
        検査:
        - 介入変数の正しい設定
        - 非介入変数の保持
        - バックドアパスの遮断
        """
        # do(x) での因果効果を計算
        causal_effect = self.compute_causal_effect(
            treatment=intervention.treatment,
            outcome=intervention.outcome,
            do_operator=intervention.do_x
        )
        
        # 介入下での d-separation を識別
        d_separated = self.compute_d_separation(
            graph=self.graph.do(intervention.treatment),
            source=intervention.outcome,
            target=intervention.treatment
        )
        
        # バックドア基準を検証
        backdoor_satisfied = self.check_backdoor_criterion(
            treatment=intervention.treatment,
            outcome=intervention.outcome
        )
        
        return {
            "causal_effect": causal_effect,
            "d_separation": d_separated,
            "backdoor_satisfied": backdoor_satisfied,
            "intervention_valid": backdoor_satisfied
        }
```

### 10.2 因果論理のモデル検査

```python
class CausalModelChecker:
    """
    因果論理のモデル検査器
    
    統合:
    - CTL (Computation Tree Logic)
    - 因果時相論理
    """
    
    # 因果時相演算子
    CAUSAL_TEMPORAL_OPERATORS = {
        "AX": "すべての未来パスの次の状態",
        "EX": "ある未来パスの次の状態",
        "AF": "すべての未来パスの最終状態",
        "EF": "ある未来パスの最終状態",
        "AG": "すべての未来パスで保持",
        "EG": "ある未来パスで保持",
        # 因果固有の演算子
        "CAUSE": "X が Y を原因（直接因果）",
        "ENABLE": "X が Y を可能にする（可能因果）",
        "PREVENT": "X が Y を阻止（負因果）"
    }
    
    def verify_causal_property(self, model, formula):
        """
        因果特性を検証
        
        例:
        - AG(do(X) → EF(Y)): X ならば、最終的に Y がある
        - AG(Cause(X,Y) → AF(Y)): X が Y を原因 поэтому Y は最終的に発生する
        """
        parsed_formula = self.parse_formula(formula)
        
        if parsed_formula.operator in self.CAUSAL_TEMPORAL_OPERATORS:
            return self.evaluate_causal_temporal(model, parsed_formula)
        else:
            return self.evaluate_standard_temporal(model, parsed_formula)
    
    def evaluate_causal_temporal(self, model, formula):
        """
        因果時相公式を評価
        
        実装:
        - 因果グラフ到達可能性分析
        - 介入効果計算
        - do-calculus モデル検査
        """
        if formula.operator == "CAUSE":
            # 因果関係を検証
            treatment_effect = self.compute_causal_effect(
                model,
                formula.cause,
                formula.effect
            )
            
            return treatment_effect > 0 and self.verify_causal_path(
                model,
                formula.cause,
                formula.effect
            )
        
        elif formula.operator == "ENABLE":
            # 可能関係を検証
            return self.check_enable_relation(
                model,
                formula.enabler,
                formula.enabled
            )
        
        return False
```

### 10.3 不変量と因果不変性

```python
class CausalInvariantVerifier:
    """
    因果不変性検証器
    
    確保:
    - 介入下の不変量の保持
    - 因果メカニズムの安定性
    """
    
    def verify_causal_invariant(self, causal_model, invariant, intervention=None):
        """
        因果不変性を検証
        
        不変量タイプ:
        - 絶対不変量: AG(P) - すべての条件下で保持
        - 因果不変量: do(X) → AG(P) - 特定の介入下で保持
        - 条件不変量: Z → AG(P) - 与えられた条件下で保持
        """
        
        if intervention is None:
            # 絶対不変量
            return self.verify_absolute_invariant(causal_model, invariant)
        else:
            # 因果不変量
            intervened_model = causal_model.do(intervention.variable, intervention.value)
            return self.verify_absolute_invariant(intervened_model, invariant)
    
    def find_causal_invariants(self, causal_model, target_variable):
        """
        因果不変量を検出
        
        方法:
        1. すべての可能な介入を列挙
        2. 介入下で不変を保つ変数を識別
        3. 因果不変量として形式化
        """
        invariants = []
        
        for intervention in self.enumerate_interventions(causal_model):
            intervened_model = causal_model.do(intervention)
            
            for variable in causal_model.variables:
                if variable == intervention.variable:
                    continue
                
                if self.verify_causal_invariant(
                    causal_model,
                    f"{variable} = {intervened_model[variable]}",
                    intervention
                ):
                    invariants.append(CausalInvariant(
                        variable=variable,
                        condition=intervention,
                        statement=f"do({intervention.variable}) → {variable} = {intervened_model[variable]}"
                    ))
        
        return invariants
    
    def verify_causal_mechanism_stability(self, causal_model, mechanism):
        """
        因果メカニズムの安定性を検証
        
        検査:
        - メカニズム関数の連続性
        - 摂動へのロバスト性
        - 跨環境の一貫性
        """
        # 1. 関数連続性
        continuity = self.check_function_continuity(mechanism.function)
        
        # 2. 局所的感受性
        sensitivity = self.compute_local_sensitivity(mechanism)
        
        # 3. ノイズ安定性
        noise_stability = self.verify_noise_stability(mechanism)
        
        return {
            "continuity": continuity,
            "sensitivity": sensitivity,
            "noise_stability": noise_stability,
            "is_stable": continuity and noise_stability
        }
```

---

## 論理的整合性エンジン宣言

> 本モジュールは Truth-OS のコア整合性保障システムである。古典論理、三値論理と量子論理を組み合わせることで、知識体系の内的整合性を確保する。

**依存モジュール**：
- EPISTEMOLOGY_AXIOMS.md（公理定義）
- DIVERGENCE_DETECTOR.md（発散検出）
- PROVENANCE_CHAIN.md（溯源管理）

**バージョン**：v2.3  
**更新概要**：ベイズ推論の進展（Transformers コンテキストベイズ推論）、形式的検証と因果推論の統合モジュールを新規追加した。
