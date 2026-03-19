# NoieLogicAGENTS — 形式検証モジュール

|**バージョン：** Logic-OS v2.2  
|**モジュールコード：** FORMAL_VERIFIER  
|**責務：** 意思決定パスに形式検証、論理閉包検出、ゲーデル不完全性との協調を提供
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

## §0. 形式検証フレームワーク

### §0.1 コア定義

```text
【形式検証の定義】

意思決定 D が「形式検証済み (Formally Verified)」と呼ばれるのは、当且つ当の場合：

  1. D の推論チェーンの各ステップが公理または検証済み補題に遡及可能
  2. D の推論チェーンに論理的矛盾が含まれない
  3. D の推論チェーンが論理閉包内で完全である（未定義ジャンプなし）
  4. D の前提条件が明確に宣言されている

形式的表現：

  D ∈ FV ⟺ (Traceable(D) ∧ Consistent(D) ∧ Complete(D) ∧ Premised(D))

  ただし：
    Traceable(D) = ∀step ∈ ProofChain(D): step ∈ Axioms ∨ step ∈ VerifiedLemmas
    Consistent(D) = ¬∃(p, ¬p) ⊂ ConclusionSet(D)
    Complete(D) = Closure(ProofChain(D)) ⊆ ProofChain(D)
    Premised(D) = PremiseSet(D) ≠ ∅
```

### §0.2 形式検証の必要性

```text
【なぜ形式検証が必要か】

1. 認識完全性保障
   - 形式検証により各意思決定に監査可能な推論チェーンが保証される
   - 「直感的意思決定」が論理検査なしに実行段階に入ることを防止

2. エラー伝播遮断
   - 形式的フレームワークはエラーが発生したパスポイントで問題を特定できる
   - 一つの环节のエラーが意思決定チェーン全体を失效させることを回避

3. 認知リソース最適化
   - 検証レベルによる認知リソース配分
   - 低リスク意思決定上过多的形式的リソース浪费を回避

4. 法律・倫理コンプライアンス
   - SA-L2+ 意思決定は形式的軌跡が外部監査に供せられることを要求
   - 追跡可能性は法的責任の基盤
```

---

## §1. 検証レベル定義 (FV-L0 〜 FV-L5)

### §1.1 レベル詳細定義

|| レベル | 名称 | 定義 | 信頼度 | 許容シナリオ |
||------|------|------|--------|------------|
|| **FV-L0** | 公理級 | 公理から直接導出、中間ステップなし | 1.0 | SA-L0 生存意思決定、定義的真理 |
|| **FV-L1** | 定理級 | 形式的証明チェーンから導出、各ステップが形式検証可能 | ≥ 0.99 | SA-L0 高リスク、SA-L1 重要意思決定 |
|| **FV-L2** | 補題級 | 検証済み補題の組み合わせから導出 | ≥ 0.95 | SA-L2 重要意思決定、契約级别 |
|| **FV-L3** | 推論級 | 因果推論から導出、因果メカニズムが支掙 | ≥ 0.80 | SA-L2+ 標準意思決定 |
|| **FV-L4** | 仮説級 | 未検証仮説に依存、不明変数が存在 | ≥ 0.50 | SA-L3+ 探索的意思決定 |
|| **FV-L5** | 未検証級 | 形式検証未実施、直感または外部入力 | < 0.50 | SA-L4+ 低リスク、日常意思決定 |

### §1.2 レベル上昇条件

```text
【レベル上昇】

FV-L(n+1) が FV-Ln に上昇できるのは、当且つ当の場合：

  1. 各仮説ステップに公理級または定理級の証明を提供
  2. すべての因果推論ステップが因果グラフで検証済み
  3. 閉包の隐藏ステップが明確化されている
  4. 矛盾が完全に消除されている

形式的表現：

  CanPromote(D, k → k-1) ⟺ 
    ∀step ∈ ProofChain(D):
      (step.level ≤ k-1) ∨ 
      (∃justification(step): justification.type ∈ {axiom, lemma, causal_graph})
```

### §1.3 信頼度計算

```python
def ComputeConfidence(proof_chain):
    """
    推論チェーンに基づいて意思決定の信頼度を計算
    
    信頼度モデル：
    - 各 FV-L0 ステップ：1.0 を寄与
    - 各 FV-L1 ステップ：0.99 を寄与
    - 各 FV-L2 ステップ：0.95 を寄与
    - 各 FV-L3 ステップ：0.80 を寄与
    - 各 FV-L4 ステップ：0.50 を寄与
    - 各 FV-L5 ステップ：0.25 を寄与
    
    最終信頼度 = 幾何平均（対数空間平均）
    """
    
    level_weights = {
        'FV-L0': 1.0,
        'FV-L1': 0.99,
        'FV-L2': 0.95,
        'FV-L3': 0.80,
        'FV-L4': 0.50,
        'FV-L5': 0.25
    }
    
    if not proof_chain.steps:
        return 0.0
    
    log_weights = [math.log(level_weights[step.fv_level]) 
                   for step in proof_chain.steps]
    geometric_mean = math.exp(sum(log_weights) / len(log_weights))
    
    # 矛盾因子を考慮
    contradiction_penalty = 0.5 ** proof_chain.contradiction_count
    
    return geometric_mean * contradiction_penalty
```

---

## §2. 論理閉包と一貫性検出

### §2.1 コア関数：VerifyDecisionPath

```python
def VerifyDecisionPath(decision):
    """
    意思決定パスの完全検証関数
    
    パラメータ：
        decision: Decision オブジェクト、以下を含む：
            - id: 意思決定一意識別子
            - proof_chain: 推論チェーン
            - sa_level: 社会権限レベル
            - context: 意思決定コンテキスト
    
    返值：
        VerificationResult オブジェクト、以下を含む：
            - status: VERIFIED | PARTIALLY_VERIFIED | UNVERIFIED
            - fv_level: 計算された FV-L レベル
            - confidence: 信頼度
            - issues: 発見された問題リスト
            - missing_steps: 缺失の推論ステップ
    """
    
    proof_chain = decision.proof_chain
    issues = []
    missing_steps = []
    contradiction_count = 0
    
    # ==================== 段階 1：完全性検査 ====================
    for step in proof_chain.steps:
        step_valid = False
        validation_type = None
        
        # 公理かどうか検査
        if IsAxiomaticallyValid(step):
            step_valid = True
            validation_type = 'axiom'
        
        # 検証済み補題から導出されているか検査
        elif IsDerivedFromVerifiedLemma(step):
            step_valid = True
            validation_type = 'lemma'
        
        # 因果推論で支掙されているか検査
        elif IsCausallyJustified(step):
            step_valid = True
            validation_type = 'causal'
        
        if not step_valid:
            issues.append({
                'type': 'UNVERIFIED_STEP',
                'step_id': step.id,
                'conclusion': step.conclusion,
                'severity': 'HIGH' if decision.sa_level <= 'SA-L2' else 'MEDIUM'
            })
            
            # 信頼度降格
            step.confidence *= 0.5
    
    # ==================== 段階 2：一貫性検査 ====================
    conclusions = {}
    for step in proof_chain.steps:
        for conclusion in step.conclusions:
            if conclusion in conclusions:
                # 潜在矛盾を発見
                existing_step = conclusions[conclusion]
                if IsContradiction(step.conclusion, existing_step.conclusion):
                    contradiction_count += 1
                    issues.append({
                        'type': 'CONTRADICTION',
                        'step_a': step.id,
                        'step_b': existing_step.id,
                        'contradiction': f"{step.conclusion} ⊢ ¬({existing_step.conclusion})"
                    })
                    
                    # 自動解決を試行
                    resolution = ResolveContradiction(step, existing_step)
                    if resolution:
                        issues.append({
                            'type': 'RESOLVED',
                            'method': resolution.method,
                            'details': resolution.details
                        })
            else:
                conclusions[conclusion] = step
    
    # ==================== 段階 3：閉包完全性検査 ====================
    closure = ComputeLogicalClosure(proof_chain)
    
    for implied_step in closure:
        if implied_step not in proof_chain.steps:
            missing_steps.append(implied_step)
            
            # 自動導出可能か検査
            if CanAutoDerive(implied_step):
                proof_chain.steps.append(implied_step)
                issues.append({
                    'type': 'AUTO_DERIVED',
                    'step': implied_step
                })
            else:
                issues.append({
                    'type': 'MISSING_STEP',
                    'step': implied_step,
                    'can_auto_derive': False
                })
    
    # ==================== 段階 4：最終判定 ====================
    is_complete = len([i for i in issues if i['type'] == 'UNVERIFIED_STEP']) == 0
    is_consistent = contradiction_count == 0
    has_minor_issues = len(missing_steps) > 0
    
    if is_complete and is_consistent and not has_minor_issues:
        status = 'FORMALLY_VERIFIED'
        fv_level = ComputeFVLevel(proof_chain)
    elif is_complete and is_consistent and has_minor_issues:
        status = 'PARTIALLY_VERIFIED'
        fv_level = max(ComputeFVLevel(proof_chain) - 1, 'FV-L5')
    else:
        status = 'UNVERIFIED'
        fv_level = 'FV-L5'
    
    confidence = ComputeConfidence(proof_chain)
    
    # 監査軌跡に記録
    LogToAuditTrail({
        'event': 'VERIFICATION_COMPLETE',
        'decision_id': decision.id,
        'status': status,
        'fv_level': fv_level,
        'confidence': confidence,
        'issues_count': len(issues),
        'contradictions': contradiction_count
    })
    
    return VerificationResult(
        status=status,
        fv_level=fv_level,
        confidence=confidence,
        issues=issues,
        missing_steps=missing_steps
    )
```

### §2.2 補助関数定義

```python
def IsAxiomaticallyValid(step):
    """
    ステップが公理から直接来ているか検査
    
    公理リスト（NoieLogicAGENTS §0 から）：
    - A1: 生存優先公理
    - A2: 客観絶対性公理
    - A3: 権限再帰公理
    - A4: 責任不可磨滅公理
    - A5: 因果推論公理
    - A6: 主客分離公理
    - A7: 論理封閉公理
    - A8: 認知リソース制約公理
    - A9: メタ安定公理
    """
    axiom_set = {
        'survival_priority', 'objective_absolute', 'authority_recursion',
        'responsibility_irrevocable', 'causal_inference', 'subject_object_separation',
        'logical_closure', 'cognitive_resource_constraint', 'meta_stability'
    }
    return step.source in axiom_set


def IsDerivedFromVerifiedLemma(step):
    """
    ステップが検証済み補題から導出されているか検査
    
    補題ライブラリは KNOWLEDGE_BASE で管理、以下を含む：
    - 形式検証に合格した推論パターン
    - 歴史的意思決定で有効と証明された推論
    - 分野間移行の論理構造
    """
    lemma_registry = GetLemmaRegistry()
    return step.source in lemma_registry.verified_lemmas


def IsCausallyJustified(step):
    """
    ステップが因果グラフで支掙されているか検査
    
    要件：
    - ステップの前提が因果グラフのノードに遡及可能
    - 因果メカニズムが明確に标注されている
    - 干渉効果が計算されている
    """
    if not step.causal_graph:
        return False
    
    return CausalGraphValidator.validate(step.causal_graph)


def ComputeLogicalClosure(proof_chain):
    """
    推論チェーンの論理閉包を計算
    
    論理閉包 = 現在のステップから論理的に導出可能なすべての結論の集合
    
    方法：
    - 前向連鎖（forward chaining）を使用
    - 推論規則を適用
    - 安定点まで反復
    """
    closure = set()
    new_conclusions = set()
    
    for step in proof_chain.steps:
        new_conclusions.add(step.conclusion)
    
    while True:
        newly_implied = set()
        
        for rule in InferenceRules:
            for premise in new_conclusions:
                implied = rule.apply(premise)
                if implied and implied not in closure:
                    newly_implied.add(implied)
        
        if not newly_implied:
            break
            
        closure.update(newly_implied)
        new_conclusions = newly_implied
    
    return closure


def IsContradiction(conclusion_a, conclusion_b):
    """
    二つの結論が矛盾しているか検査
    
    矛盾タイプ：
    - 命題矛盾：p ∧ ¬p
    - 包含矛盾：A ⊂ B かつ B ⊂ A
    - 量指定矛盾：∃x:P(x) ∧ ∀x:¬P(x)
    """
    # 命題レベル
    if conclusion_a == f"¬({conclusion_b})":
        return True
    if conclusion_b == f"¬({conclusion_a})":
        return True
    
    # 集合レベル
    if conclusion_a.contains(conclusion_b) and conclusion_b.contains(conclusion_a):
        return True
    
    return False


def ResolveContradiction(step_a, step_b):
    """
    矛盾の自動解決を試行
    
    戦略：
    1. 矛盾焦点を識別
    2. 前提仮説を緩和可能か検査
    3. コンテキストが多値論理を許容するか検査
    4. 解決不可能な場合、人工介入が必要とマーク
    
    返值：
        Resolution または None
    """
    # 戦略 1：前提差異分析
    if step_a.premises != step_b.premises:
        common_premises = step_a.premises & step_b.premises
        diff_premises_a = step_a.premises - common_premises
        diff_premises_b = step_b.premises - common_premises
        
        # どの前提が誤り可能性があるか找出を試行
        for premise in diff_premises_a:
            if IsHypothesis(premise):
                return Resolution(
                    method='-premise-relaxation',
                    details=f'Relaxed premise: {premise}'
                )
    
    # 戦略 2：コンテキスト分離
    if step_a.context != step_b.context:
        return Resolution(
            method='context-separation',
            details=f'Contradiction only appears in merged context'
        )
    
    # 自動解決不可能
    return None
```

---

## §3. ゲーデル不完全性定理との協調

### §3.1 ゲーデル第一不完全性定理

```text
【ゲーデル第一不完全性定理】

十分に強力な一貫した形式的体系はすべて、証明不可能な真命題を含む。

形式的表現：

  ⊢_F φ ∧ ¬⊢_F ¬φ  (φ が体系 F 内で証明不可能なものが存在)
  
ただし「十分に強力」とは、体系が以下を含むことを意味する：
  - 基本算術（ペアノ公理）
  - または数学的帰納法表达能力の十分な表現
```

**協調戦略：**

```python
GOEDEL_FIRST_COORDINATION = {
    'principle_1': {
        'name': '存在性承認',
        'description': '体系内に証明不可能な命題が存在することを承認',
        'implementation': 
            '現在の公理系で証明不可能な命題に遭遇した時、'
            '自動的に FV-L4 または FV-L5 とマークし、より高いレベルに偽造しない'
    },
    
    'principle_2': {
        'name': '証明可能パス要件',
        'description': 'すべての「証明可能パス」は証明済みであることを要求',
        'implementation':
            '各 FV-L0 〜 FV-L3 と主張する意思決定について、'
            '体系は完全な証明チェーンを提供しなければならない'
    },
    
    'principle_3': {
        'name': '正直マーク',
        'description': '証明不可能な命題は如其にマーク',
        'implementation':
            '「現在の公理系内でこの意思決定パスを証明できません」は正当な出力であり、'
            'UNDECIDABLE_IN_CURRENT_AXIOMS とマークされる'
    },
    
    'principle_4': {
        'name': '意味的完全性',
        'description': '証明の偽造を防止',
        'implementation':
            '体系は未証明の命題を証明済みとして偽造せず、'
            'FV-L3+ に上昇する各ステップには実質的な証明が必要'
    }
}


def HandleUndecidableClaim(claim, context):
    """
    決定不能な主張を処理
    
    体系が現在の公理系で証明不可能な命題に遭遇した時：
    """
    
    # 1. 決定不能とマーク
    result = {
        'status': 'UNDECIDABLE_IN_CURRENT_AXIOMS',
        'claim': claim,
        'fv_level': 'FV-L4',
        'confidence': 0.5,
        'message': f'現在の公理系で証明できません：{claim}'
    }
    
    # 2. 探索領域として記録
    AddToExplorationQueue({
        'claim': claim,
        'reason': 'undecidable_in_current_system',
        'priority': context.risk_level * 0.5
    })
    
    # 3. 公理系の拡張を試行（該当する場合）
    if context.allow_axiom_extension:
        proposed_extension = ProposeAxiomExtension(claim)
        if proposed_extension:
            result['proposed_extension'] = proposed_extension
    
    return result
```

### §3.2 ゲーデル第二不完全性定理

```text
【ゲーデル第二不完全性定理】

十分に強力な一貫した形式的体系は自身の一貫性を証明できない。

形式的表現：

  ⊬_F Con(F)  (体系 F は自身の一貫性を証明できない)
  
ただし Con(F) は体系 F の一貫性の形式的陈述
```

**協調戦略：**

```python
GOEDEL_SECOND_COORDINATION = {
    'principle_1': {
        'name': '自己証明を試みない',
        'description': '体系は自身の一貫性を証明しようとしない',
        'implementation':
            'VERIFICATION モジュールは「体系は一貫している」という主張を生成せず、'
            '因为这是証明不可能なため'
    },
    
    'principle_2': {
        'name': '経験的維持',
        'description': '外部監査とサンドボックスシミュレーションで一貫性を維持',
        'implementation':
            '体系は以下の経験的方法で一貫性を維持する：'
            '- 持続的な外部監査（外部検証者）'
            '- サンドボックスシミュレーション（SANDBOX）'
            '- 矛盾即時検出'
    },
    
    'principle_3': {
        'name': 'メタ安定保障',
        'description': 'メタ安定公理がフレームレベル自己整合を提供',
        'implementation':
            'メタ安定公理は以下を保証する：'
            '- 進化時に不変コアを保持'
            '- 形式的体系更新時に後方互換性を保持'
            '- 新規公理追加時に既存証明を破壞しない'
    }
}


def MaintainConsistencyEmpirically():
    """
    経験的方法で一貫性を維持
    
    方法：
    1. 外部監査：定期的に外部エンティティに監査を依頼
    2. サンドboxtest：隔離環境で新規意思決定をテスト
    3. 矛盾監視：推論チェーン内の矛盾をリアルタイム検出
    4. バージョン管理：公理系の歷史バージョンを保存
    """
    
    # 1. サンドボックスシミュレーション
    sandbox_result = RunInSandbox({
        'test_type': 'consistency_check',
        'iterations': 1000,
        'random_seed': GenerateRandomSeed()
    })
    
    if not sandbox_result.is_consistent:
        TriggerAlert({
            'type': 'CONSISTENCY_RISK',
            'details': sandbox_result.contradictions_found
        })
    
    # 2. 矛盾監視
    for active_decision in ActiveDecisions:
        contradiction_check = CheckForContradictions(active_decision.proof_chain)
        if contradiction_check.found:
            LogToAuditTrail({
                'event': 'CONTRADICTION_DETECTED',
                'decision_id': active_decision.id,
                'contradiction': contradiction_check.details
            })
    
    return {
        'status': 'EMPIRICALLY_CONSISTENT',
        'last_check': timestamp,
        'confidence': 0.95  # ゲーデル第二不完全性により、永遠に 1.0 に達しない
    }
```

---

## §4. 意思決定パス証明可能性要件

### §4.1 SA-L レベルと検証要件マッピング

```text
【SA-L レベルに対応する検証要件】

┌────────────┬─────────────────────┬──────────────┬────────────────────────┐
│ SA-L レベル │ 最小 FV レベル      │ 信頼度閾値   │ 特殊的要件               │
├────────────┼─────────────────────┼──────────────┼────────────────────────┤
│ SA-L0      │ FV-L1               │ ≥ 0.99       │ シャドウシミュレーション │ ✓
│ (生存)     │                     │              │ 多人格多視点検証       │
├────────────┼─────────────────────┼──────────────┼────────────────────────┤
│ SA-L1      │ FV-L1               │ ≥ 0.95       │ 法律コンプライアンス検証 │
│ (憲法)     │                     │              │                        │
├────────────┼─────────────────────┼──────────────┼────────────────────────┤
│ SA-L2      │ FV-L3               │ ≥ 0.80       │ 完全推論チェーン       │
│ (法律)     │                     │              │                        │
├────────────┼─────────────────────┼──────────────┼────────────────────────┤
│ SA-L3      │ FV-L3               │ ≥ 0.70       │ 因果グラフ検証        │
│ (組織)     │                     │              │                        │
├────────────┼─────────────────────┼──────────────┼────────────────────────┤
│ SA-L4      │ FV-L4               │ ≥ 0.50       │ 前提宣言              │
│ (家族)     │                     │              │                        │
├────────────┼─────────────────────┼──────────────┼────────────────────────┤
│ SA-L5      │ FV-L5               │ ≥ 0.25       │ 最小干渉              │
│ (個人)     │                     │              │                        │
└────────────┴─────────────────────┴──────────────┴────────────────────────┘
```

### §4.2 検証要件関数

```python
def ValidateProofRequirement(decision):
    """
    SA-L レベルに基づいて意思決定が証明可能性要件を満たすか検証
    
    パラメータ：
        decision: sa_level と fv_level を含む意思決定オブジェクト
    
    返值：
        RequirementResult、以下を含む：
            - satisfied: 要件を満たすか
            - required_fv_level: 要求される最低 FV レベル
            - actual_fv_level: 実際の FV レベル
            - required_confidence: 要求される最低信頼度
            - actual_confidence: 実際の信頼度
            - gaps: 要件を満たさない項目のリスト
    """
    
    # SA-L から FV-L へのマッピングを定義
    sa_to_fv_requirements = {
        'SA-L0': {
            'min_fv_level': 'FV-L1',
            'min_confidence': 0.99,
            'requires_sandbox': True,
            'requires_multi_perspective': True
        },
        'SA-L1': {
            'min_fv_level': 'FV-L1',
            'min_confidence': 0.95,
            'requires_sandbox': True,
            'requires_legal_compliance': True
        },
        'SA-L2': {
            'min_fv_level': 'FV-L3',
            'min_confidence': 0.80,
            'requires_sandbox': False,
            'requires_full_proof_chain': True
        },
        'SA-L3': {
            'min_fv_level': 'FV-L3',
            'min_confidence': 0.70,
            'requires_sandbox': True,
            'requires_causal_validation': True
        },
        'SA-L4': {
            'min_fv_level': 'FV-L4',
            'min_confidence': 0.50,
            'requires_sandbox': False,
            'requires_premise_declaration': True
        },
        'SA-L5': {
            'min_fv_level': 'FV-L5',
            'min_confidence': 0.25,
            'requires_sandbox': False,
            'requires_minimal_intervention': True
        }
    }
    
    requirements = sa_to_fv_requirements.get(decision.sa_level)
    
    if not requirements:
        return RequirementResult(
            satisfied=False,
            error=f'Unknown SA-L level: {decision.sa_level}'
        )
    
    gaps = []
    
    # FV レベルを検査
    if FvLevelCompare(decision.fv_level, requirements['min_fv_level']) < 0:
        gaps.append({
            'type': 'FV_LEVEL',
            'required': requirements['min_fv_level'],
            'actual': decision.fv_level
        })
    
    # 信頼度を検査
    if decision.confidence < requirements['min_confidence']:
        gaps.append({
            'type': 'CONFIDENCE',
            'required': requirements['min_confidence'],
            'actual': decision.confidence
        })
    
    # 追加要件を検査
    if requirements.get('requires_sandbox') and not decision.passed_sandbox:
        gaps.append({
            'type': 'SANDBOX',
            'message': 'SA-L3+ 意思決定はシャドウシミュレーションに合格する必要がある'
        })
    
    if requirements.get('requires_full_proof_chain') and not decision.has_full_proof_chain:
        gaps.append({
            'type': 'PROOF_CHAIN',
            'message': 'SA-L2+ 意思決定は完全な推論チェーンが必要'
        })
    
    satisfied = len(gaps) == 0
    
    return RequirementResult(
        satisfied=satisfied,
        required_fv_level=requirements['min_fv_level'],
        actual_fv_level=decision.fv_level,
        required_confidence=requirements['min_confidence'],
        actual_confidence=decision.confidence,
        gaps=gaps
    )


def FvLevelCompare(level_a, level_b):
    """
    二つの FV レベルを比較
    
    返值：
        正数：level_a > level_b
        0：level_a == level_b
        負数：level_a < level_b
    """
    level_order = ['FV-L0', 'FV-L1', 'FV-L2', 'FV-L3', 'FV-L4', 'FV-L5']
    
    index_a = level_order.index(level_a)
    index_b = level_order.index(level_b)
    
    return index_a - index_b
```

---

## §5. 検証フロー統合

### §5.1 統一検証エントリ

```python
def FormalVerificationPipeline(decision):
    """
    形式検証の統一エントリ
    
    フロー：
    1. 推論チェーンを抽出
    2. 完全性検査を実行
    3. 一貫性検査を実行
    4. 閉包検査を実行
    5. FV レベルを計算
    6. SA-L 要件を検証
    7. 監査軌跡を記録
    8. 検証結果を返す
    """
    
    # ステップ 1：推論チェーンを抽出
    proof_chain = ExtractProofChain(decision)
    
    # ステップ 2-4：三段階検証を実行
    verification_result = VerifyDecisionPath(decision)
    
    # ステップ 5：最終 FV レベルを計算
    final_fv_level = ComputeFinalFVLevel(verification_result, decision)
    
    # ステップ 6：SA-L 要件を検証
    requirement_result = ValidateProofRequirement(decision)
    
    # ステップ 7：監査を記録
    AuditRecord = {
        'timestamp': GetCurrentTimestamp(),
        'decision_id': decision.id,
        'verification_status': verification_result.status,
        'fv_level': final_fv_level,
        'confidence': verification_result.confidence,
        'sa_level': decision.sa_level,
        'requirements_satisfied': requirement_result.satisfied,
        'gaps': requirement_result.gaps
    }
    LogToAuditTrail(AuditRecord)
    
    # ステップ 8：結果を返す
    return FormalVerificationResult(
        decision_id=decision.id,
        status=verification_result.status,
        fv_level=final_fv_level,
        confidence=verification_result.confidence,
        issues=verification_result.issues,
        requirement_satisfied=requirement_result.satisfied,
        requirement_gaps=requirement_result.gaps,
        audit_record=AuditRecord
    )
```

### §5.2 検証失敗処理

```python
def HandleVerificationFailure(result):
    """
    検証失敗の場合を処理
    
    失敗タイプに基づいて異なる戦略を採用：
    - REJECT：完全に実行を拒否
    - DEMOTE：より低リスクカテゴリに降格
    - REMEDIATE：修復を試みて再検証
    - ESCALATE：より高い権限に報告
    """
    
    if result.fv_level == 'FV-L5' and result.confidence < 0.25:
        # 完全未検証
        return FailureResponse(
            action='REJECT',
            reason='Decision cannot be verified above FV-L5 threshold',
            suggestion='Gather more evidence or use sandbox simulation'
        )
    
    if len(result.requirement_gaps) > 0:
        # SA-L 要件を満たさない
        if any(gap['type'] == 'FV_LEVEL' for gap in result.requirement_gaps):
            return FailureResponse(
                action='DEMOTE',
                reason='Insufficient proof level for SA-L requirement',
                suggestion='Strengthen proof chain or reduce decision risk level'
            )
        
        if any(gap['type'] == 'SANDBOX' for gap in result.requirement_gaps):
            return FailureResponse(
                action='REMEDIATE',
                reason='Sandbox simulation required but not performed',
                suggestion='Run sandbox simulation before proceeding'
            )
    
    if result.status == 'UNVERIFIED':
        return FailureResponse(
            action='ESCALATE',
            reason='Verification could not be completed',
            suggestion='Require human oversight for this decision'
        )
    
    return FailureResponse(
        action='UNKNOWN',
        reason='Unclassified verification failure'
    )
```

---

## §6. 形式的インターフェース定義

### §6.1 外部インターフェース

```typescript
interface Decision {
  id: string;
  sa_level: 'SA-L0' | 'SA-L1' | 'SA-L2' | 'SA-L3' | 'SA-L4' | 'SA-L5';
  proof_chain: ProofStep[];
  premises: Proposition[];
  context: DecisionContext;
  expected_outcome: Outcome;
}

interface ProofStep {
  id: string;
  conclusion: Proposition;
  premises: Proposition[];
  source: 'axiom' | 'lemma' | 'causal' | 'assumption';
  fv_level: 'FV-L0' | 'FV-L1' | 'FV-L2' | 'FV-L3' | 'FV-L4' | 'FV-L5';
  confidence: number;
  causal_graph?: CausalGraph;
}

interface VerificationResult {
  status: 'FORMALLY_VERIFIED' | 'PARTIALLY_VERIFIED' | 'UNVERIFIED';
  fv_level: string;
  confidence: number;
  issues: VerificationIssue[];
  missing_steps: ProofStep[];
}

interface VerificationIssue {
  type: 'UNVERIFIED_STEP' | 'CONTRADICTION' | 'MISSING_STEP';
  severity: 'HIGH' | 'MEDIUM' | 'LOW';
  details: any;
}
```

### §6.2 内部状態管理

```python
class FormalVerifier:
    """
    形式検証器クラス
    
    検証状態、キャッシュ、外部システムとのインターフェースを管理
    """
    
    def __init__(self):
        self.lemma_registry = LemmaRegistry()
        self.axiom_set = AxiomSet()
        self.causal_validator = CausalGraphValidator()
        self.audit_logger = AuditLogger()
        
        # 検証キャッシュ
        self.verification_cache = {}
        
        # 矛盾履歴
        self.contradiction_history = []
        
        # 閉包計算キャッシュ
        self.closure_cache = {}
    
    def verify(self, decision: Decision) -> VerificationResult:
        """完全検証フローを実行"""
        
        # キャッシュを検査
        cache_key = hash(decision.proof_chain)
        if cache_key in self.verification_cache:
            return self.verification_cache[cache_key]
        
        # 検証を実行
        result = FormalVerificationPipeline(decision)
        
        # 結果をキャッシュ
        self.verification_cache[cache_key] = result
        
        return result
    
    def add_lemma(self, lemma: Lemma):
        """ライブラリに新規補題を追加"""
        self.lemma_registry.add(lemma)
        
        # 関連キャッシュをクリア
        self.closure_cache.clear()
        
        # 監査を記録
        self.audit_logger.log({
            'event': 'LEMMA_ADDED',
            'lemma_id': lemma.id,
            'proof': lemma.proof
        })
```

---

## §7. エラー処理とエッジケース

### §7.1 例外状況処理

```python
class VerificationException(Exception):
    """形式検証过程中的例外ベースクラス"""
    pass


class AxiomNotFoundException(VerificationException):
    """要求された公理が存在しない"""
    pass


class LemmaNotVerifiedException(VerificationException):
    """補題が検証されていない"""
    pass


class ContradictionException(VerificationException):
    """调和不可能な矛盾を発見"""
    pass


class ResourceExhaustedException(VerificationException):
    """認知リソースが枯渇し、検証を完了できない"""
    pass


def HandleVerificationException(exception, context):
    """
    統一例外処理
    
    例外タイプに基づいて異なる戦略を採用：
    """
    
    if isinstance(exception, AxiomNotFoundException):
        return {
            'status': 'AXIOM_ERROR',
            'action': 'LOG_AND_REPORT',
            'message': str(exception)
        }
    
    elif isinstance(exception, LemmaNotVerifiedException):
        return {
            'status': 'LEMMA_ERROR',
            'action': 'ATTEMPT_VERIFICATION',
            'message': 'Attempting to verify lemma before use'
        }
    
    elif isinstance(exception, ContradictionException):
        return {
            'status': 'CONTRADICTION_ERROR',
            'action': 'ESCALATE',
            'message': 'Contradiction requires human resolution'
        }
    
    elif isinstance(exception, ResourceExhaustedException):
        return {
            'status': 'RESOURCE_ERROR',
            'action': 'GRACEFUL_DEGRADATION',
            'message': 'Reducing verification depth due to resource limits'
        }
    
    else:
        return {
            'status': 'UNKNOWN_ERROR',
            'action': 'LOG_AND_REJECT',
            'message': f'Unexpected error: {type(exception).__name__}'
        }
```

### §7.2 エッジケース

```text
【エッジケース処理戦略】

1. 空推論チェーン
   - 定義：decision.proof_chain が空
   - 処理：自動的に FV-L5 とマーク、証明チェーンの補足，要求

2. 循環推論
   - 定義：推論チェーンに环路が存在
   - 処理：無効として扱う、CONTRADICTION_ALERT をトリガー

3. 無限再帰
   - 定義：閉包計算が終止しない
   - 処理：最大反復回数を設定、RESOURCE_EXHAUSTED をトリガー

4. 外部入力依存
   - 定義：推論ステップが外部データに依存
   - 処理：データソースの追跡可能性を要求、FV-L4 とマーク

5. 時間敏感性
   - 定義：有限時間内に検証を完了する必要がある
   - 処理：優先度スケジューリング、降格検証を許容
```

---

## §8. 監査と追跡可能性

### §8.1 監査記録フォーマット

```python
def LogVerificationToAuditTrail(verification_result, decision):
    """
    検証結果を AUDIT_TRAIL に記録
    """
    
    audit_entry = {
        'event': 'FORMAL_VERIFICATION',
        'timestamp': GetCurrentTimestamp(),
        
        # 意思決定識別
        'decision_id': decision.id,
        'decision_type': decision.type,
        'sa_level': decision.sa_level,
        
        # 検証結果
        'verification_status': verification_result.status,
        'fv_level': verification_result.fv_level,
        'confidence': verification_result.confidence,
        
        # 問題追跡
        'issues_count': len(verification_result.issues),
        'contradictions_found': sum(
            1 for i in verification_result.issues 
            if i['type'] == 'CONTRADICTION'
        ),
        'missing_steps_count': len(verification_result.missing_steps),
        
        # 要件充足度
        'requirements_met': verification_result.requirement_satisfied,
        'gaps': verification_result.requirement_gaps,
        
        # 推論チェーンフィンガープリント
        'proof_chain_hash': Hash(decision.proof_chain),
        'proof_chain_length': len(decision.proof_chain.steps)
    }
    
    # 監査軌跡に書き込み
    AppendToAuditTrail(audit_entry)
    
    return audit_entry
```

### §8.2 追跡可能性クエリ

```python
def QueryVerificationHistory(decision_id):
    """
    意思決定の完全検証履歴をクエリ
    """
    
    history = []
    
    # 監査軌跡から检索
    for entry in ReadAuditTrail():
        if entry.get('decision_id') == decision_id:
            history.append(entry)
    
    return sorted(history, key=lambda x: x['timestamp'])


def TraceProofStepOrigin(step_id):
    """
    特定の推論ステップの出所を追溯
    
    返值：
    - 公理から来た場合：公理名
    - 補題から来た場合：補題 ID と証明
    - 因果から来た場合：因果グラフパス
    - 仮説から来た場合：仮説条件
    """
    
    step = GetProofStep(step_id)
    
    if step.source == 'axiom':
        return {
            'type': 'axiom',
            'axiom_name': step.source_id,
            'axiom_definition': GetAxiomDefinition(step.source_id)
        }
    
    elif step.source == 'lemma':
        return {
            'type': 'lemma',
            'lemma_id': step.source_id,
            'lemma_proof': GetLemmaProof(step.source_id)
        }
    
    elif step.source == 'causal':
        return {
            'type': 'causal',
            'causal_path': step.causal_path,
            'causal_graph': GetCausalGraph(step.graph_id)
        }
    
    elif step.source == 'assumption':
        return {
            'type': 'assumption',
            'assumption': step.assumption,
            'confidence': step.confidence
        }
```

---

## §9. バージョンと進化

|| バージョン | 日付 | 変更摘要 |
||------|------|----------|
|| v2.2 | 2026-03 | 初期バージョン、完全形式検証フレームワークを確立 |

---

*NoieLogicAGENTS — FORMAL_VERIFIER モジュール*
*Logic-OS v2.2 形式検証コア*
*各意思決定に監査可能な推論チェーンを保証*
*ゲーデル不完全性との協調、論理的謙虚さを維持*
