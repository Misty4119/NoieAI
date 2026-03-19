# COUNTERFACTUAL.md

## 反事実推論フレームワーク (Counterfactual Reasoning Framework v2.2)

**定義：** 本モジュールはNoieLogicAGENTSロジックエンジンのコアサブモジュールであり、反事実推論（Counterfactual Reasoning）フレームワークを実装し、第3層因果推論（反事実）と双世界モデル計算專門担当。

**システム定位：** LOGIC_ENGINE.mdのL3詳細モジュールとして、因果推論の最高レベルである反事実推論專門担当。「もし当初違う選択をしていたら、結果はどうだったか？」という問題に答える。

**依存モジュール：** LOGIC_ENGINE.md、CAUSAL_INFERENCE.md、ABDUCTIVE_REASONING.md、CONSTRAINTS.md

---

> ⚠️ 重要安全と意思決定プロトコル (CRITICAL SAFETY & DECISION PROTOCOL v2.2):
> 1. CONSTRAINTS.mdと社会権限レベル（SA-L0〜SA-L5）を厳密に遵守する。
> 2. 因果推論：すべての意思決定は因果グラフ（DAG）に基づき、因果メカニズムを标注する。
> 3. 主客体分離：意思決定推論は自己状態と環境状態を混同してはならない。
> 4. 形式化検証：高リスクの意思決定パスは論理閉包検証を通過する必要がある。
> 5. 影子シミュレーション：SA-L3+操作涉及時は、SANDBOXで事前に結果をシミュレーションする。
> 6. 情報ビット完全性：情報ビットを作成してはならない。KNOWLEDGE_BASEが空の場合は、「データ欠損」を明確に宣言する。
> 7. 認知リソース制約：意思決定の深さは利用可能な認知リソースを超えてはならない。
> 8. 監査：すべての競合、拒否、形式検証結果はAUDIT_TRAILに記録する。
> 9. 生存優先：すべての意思決定は実行前に吸収状態につながらないことを検証する必要がある。
> 10. 自己進化：公理系が進化する場合、不変コアは保持されなければならない。

---

## §1. 反事実推論理論基盤

### §1.1 反事実定義

反事実推論は因果推論の最高レベルであり、「もし...だったらどうなったか？」という类型の問題を処理する。

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                    反事実推論定義                                  ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  【反事実問題】                                                     ║
║                                                                       ║
║  「もし当初X而非X'をしていたら、結果Yはどうだったか？」                      ║
║                                                                       ║
║  形式化表現：                                                       ║
║    P(Y_x = y | X = x', Y = y')                                    ║
║                                                                       ║
║  解釈：                                                             ║
║    X = x'かつY = y'が観測された条件の下で、                          ║
║    Xがxに干渉されたら、Yがyになる確率                              ║
║                                                                       ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  【反事実 vs 干渉】                                                ║
║                                                                       ║
║  干渉 (Intervention):                                              ║
║    P(Y | do(X = x)) — もし私たちが今Xを強制としたら、結果はどうなる？            ║
║                                                                       ║
║  反事実 (Counterfactual):                                          ║
║    P(Y_x = y | X = x', Y = y') — もし当初違う選択をしていたら、             ║
║    現在の結果が分かっている下で、結果はどうなるか？                            ║
║                                                                       ║
║  重要な区別：                                                         ║
║    - 干渉は前瞻的（未来）                                       ║
║    - 反事实は回顧的（過去+仮説）                                ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### §1.2 三層因果推論における位置

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                    三層因果推論アーキテクチャ                                 ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  Layer 3 ────────── 反事実 (Counterfactual) ────────── 最高レベル    ║
║  ══════════════════════════════════════════════════════              ║
║  「もし当初違う選択をしていたら、結果はどうだったか？」                      ║
║  ツール：構造方程式、双世界モデル                                        ║
║  数学：P(Y_x | X=x', Y=y')                                        ║
║  実装：本モジュール                                                       ║
║                                                                       ║
║        ═══════════════════════════════════════                       ║
║                                                                       ║
║  Layer 2 ────────── 干渉 (Intervention) ────────── 中間レベル         ║
║  「もし私がXを強制的に変えたら、Yはどう変わる？」                                ║
║  ツール：do-calculus、截断分解                                       ║
║  数学：P(Y | do(X=x))                                              ║
║  実装：CAUSAL_INFERENCE.md                                          ║
║                                                                       ║
║        ═══════════════════════════════════════                       ║
║                                                                       ║
║  Layer 1 ────────── 関連 (Association) ────────── 基底レベル         ║
║  「Xが发生观测された時、Yの確率は？」                             ║
║  ツール：条件確率、ベイズ推論                                        ║
║  数学：P(Y | X)                                                    ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

---

## §2. 双世界モデル

### §2.1 双世界理論

```python
"""
双世界モデル実装
"""

from typing import Dict, List, Optional, Set, Any
from dataclasses import dataclass
from enum import Enum
import copy

@dataclass
class StructuralEquationModel:
    """
    構造方程式モデル
    
    M = (U, V, F)
    - U: 外生変数集合
    - V: 内生変数集合
    - F: 構造方程式集合
    """
    exogenous_vars: Set[str]      # U: 外部変数
    endogenous_vars: Set[str]     # V: 内部変数
    structural_equations: Dict[str, str]  # F: 構造方程式
    
    def evaluate(self, assignments: Dict[str, Any]) -> Dict[str, Any]:
        """
        構造方程式を評価
        
        現在の割り当てに基づいてすべての内生変数の値を計算
        """
        # 拓扑ソートで親ノードを先に計算
        evaluated = dict(assignments)
        
        for var in self._topological_order():
            if var in self.endogenous_vars:
                equation = self.structural_equations.get(var, "0")
                # 簡略化実装：フレームワークレベル
                evaluated[var] = self._evaluate_equation(
                    equation, evaluated
                )
        
        return evaluated
    
    def _topological_order(self) -> List[str]:
        """拓扑ソート順序を取得"""
        # 簡略化実装
        return list(self.endogenous_vars)
    
    def _evaluate_equation(self, equation: str, context: Dict) -> Any:
        """单个構造方程式を評価"""
        # フレームワーク実装
        return 0.0


@dataclass
class TwinWorld:
    """
    双世界の一つ：事実世界または反事実世界
    
    各世界には以下が含まれる：
    - 構造方程式モデル
    - 外生変数の割り当て（固定）
    - 干渉集合
    """
    sem: StructuralEquationModel
    exogenous_assignments: Dict[str, Any]  # Uの値
    interventions: Dict[str, Any]  # do(X=x) 干渉
    world_id: str  # "factual" または "counterfactual"
    
    def evaluate(self) -> Dict[str, Any]:
        """
        世界全体を評価
        
        干渉を適用してすべての内生変数を計算
        """
        # 干渉を外生割り当てにマージ
        context = dict(self.exogenous_assignments)
        context.update(self.interventions)
        
        return self.sem.evaluate(context)
    
    def get(self, variable: str) -> Any:
        """特定変数の値を取得"""
        result = self.evaluate()
        return result.get(variable)


class TwinWorldModel:
    """
    双世界モデル
    
    以下を含む：
    - 事実世界 (factual world)：実際に发生した歴史
    - 反事実世界 (counterfactual world)：仮説歴史
    """
    
    def __init__(self, sem: StructuralEquationModel):
        self.sem = sem
        self.factual_world: Optional[TwinWorld] = None
        self.counterfactual_world: Optional[TwinWorld] = None
    
    def setup_factual_world(
        self,
        observed_assignments: Dict[str, Any]
    ):
        """
        事実世界を設定
        
        観測結果から外生変数Uの値を推論
        """
        # 外生変数を逆算：観測結果からUを推論
        inferred_u = self._infer_exogenous(observed_assignments)
        
        self.factual_world = TwinWorld(
            sem=self.sem,
            exogenous_assignments=inferred_u,
            interventions={},
            world_id="factual"
        )
    
    def setup_counterfactual_world(
        self,
        treatment: str,
        treatment_value: Any
    ):
        """
        反事実世界を設定
        
        干渉do(X=x)を設定
        """
        # 事実世界と同じ外生割り当てを使用
        if self.factual_world is None:
            raise ValueError("Must setup factual world first")
        
        self.counterfactual_world = TwinWorld(
            sem=self.sem,
            exogenous_assignments=self.factual_world.exogenous_assignments,
            interventions={treatment: treatment_value},
            world_id="counterfactual"
        )
    
    def _infer_exogenous(
        self,
        observations: Dict[str, Any]
    ) -> Dict[str, Any]:
        """
        観測結果から外生変数を推論
        
        これは反事実推論の重要なステップ
        """
        # フレームワーク実装：完全前方モデルが必要
        inferred = {}
        
        # 簡略化：外生変数が観測可能または推論可能と仮定
        for var in self.sem.exogenous_vars:
            if var in observations:
                inferred[var] = observations[var]
        
        return inferred
```

### §2.2 ctf-calculusと反事実推論統合

**ctf-calculus** (Correa & Bareinboim, 2025) はDo-Calculusを反事実領域に拡張し、干渉と反事実の統一推論フレームワークを実装した。本モジュールはctf-calculusを統合して反事実計算能力を高める。

```text
╔═══════════════════════════════════════════════════════════════════════╗
║          ctf-calculus: 反事実微積分と双世界モデル統合                ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  【ctf-calculusコア思想】                                         ║
║                                                                       ║
║  従来の双世界モデルは完全な構造方程式モデル (SEM) が必要なため計算コストが高い。         ║
║  ctf-calculusは完全なSEM없이反事実推論を可能にする方法を提供する。   ║
║                                                                       ║
║  【統一フレームワーク】                                                       ║
║                                                                       ║
║  ctf-calculusの四つのルールにより：                                 ║
║                                                                       ║
║  • 観測データに基づいて反事実確率を識別                                   ║
║  • 強い仮定（確定性関数など）への依存を軽減                             ║
║  • 部分識別と感度分析を実装                                        ║
║                                                                       ║
║  【双世界モデルとの統合】                                               ║
║                                                                       ║
║  1. 識別段階：ctf-calculusルールを使用して識別可能な反事実を識別            ║
║  2. 計算段階：識別可能な場合、双世界モデルで精密計算               ║
║  3. 推定段階：部分識別の場合、信用区間を返す                           ║
║                                                                       ║
║  【優位性比較】                                                       ║
║                                                                       ║
║  ┌─────────────────┬───────────────────┬───────────────────────┐    ║
║  │     方法        │   双世界モデル       │    ctf-calculus       │    ║
║  ├─────────────────┼───────────────────┼───────────────────────┤    ║
║  │  SEM依存      │      完全依存       │     部分依存          │    ║
║  │  識別能力      │      精密識別        │  精密+部分識別        │    ║
║  │  計算コスト      │       高            │       可変            │    ║
║  │  適用シーン     │  完全既知構造        │  未知/部分構造        │    ║
║  └─────────────────┴───────────────────┴───────────────────────┘    ║
║                                                                       ║
║  【引用】                                                           ║
║  Correa, J.D., & Bareinboim, E. (2025).                           ║
║  "A Calculus for Counterfactual Reasoning."                        ║
║  ICML 2025.                                                         ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

```python
"""
ctf-calculus統合実装
強化反事実推論エンジン
"""

class CTFCalculusCounterfactualEngine:
    """
    ctf-calculusを統合した反事実エンジン
    
    双世界モデルとctf-calculusの優位性を組み合わせ
    """
    
    def __init__(self, sem: StructuralEquationModel, causal_graph: CausalGraph):
        self.sem = sem
        self.graph = causal_graph
        self.twin_world_model = TwinWorldModel(sem)
    
    def compute_counterfactual_enhanced(
        self,
        treatment: str,
        treatment_factual: Any,
        treatment_counterfactual: Any,
        outcome: str,
        observed_outcome: Any,
        observations: Dict[str, Any]
    ) -> Dict[str, Any]:
        """
        強化反事実計算
        
        優先的にctf-calculus識別を試み、失敗すれば双世界モデルを使用
        """
        
        # ═══════════════════════════════════════════════════════════
        # STEP 1: ctf-calculus識別を試行
        # ═══════════════════════════════════════════════════════════
        
        ctf_result = self._try_ctf_identification(
            treatment, treatment_counterfactual,
            outcome, observed_outcome, observations
        )
        
        if ctf_result["identifiable"]:
            return {
                "method": "ctf-calculus",
                "counterfactual_value": ctf_result["value"],
                "probability": ctf_result["probability"],
                "confidence": ctf_result["confidence"],
                "identification_method": "ctf_rules"
            }
        
        # ═══════════════════════════════════════════════════════════
        # STEP 2: 双世界モデルにフォールバック
        # ═══════════════════════════════════════════════════════════
        
        twin_result = self._compute_twin_world_counterfactual(
            treatment, treatment_counterfactual,
            outcome, observed_outcome, observations
        )
        
        return {
            "method": "twin_world",
            "counterfactual_value": twin_result["counterfactual_value"],
            "probability": twin_result["probability"],
            "confidence": twin_result["confidence"],
            "identification_method": "sem_evaluation"
        }
    
    def _try_ctf_identification(
        self,
        treatment: str,
        treatment_counterfactual: Any,
        outcome: str,
        observed_outcome: Any,
        observations: Dict[str, Any]
    ) -> Dict[str, Any]:
        """
        ctf-calculusを使用して反事実の識別を試行
        
        返回：(識別可能か, 識別値, 信心度)
        """
        # 簡略化実装：フレームワークレベル
        
        # CF1条件をチェック
        # もし(Y ⊥⊥ Z | X, W) が G_overline{X} で成立するなら
        # 計算を簡略化可能
        
        # ここに完全な条件独立性検定が必要
        
        return {
            "identifiable": False,
            "value": None,
            "probability": None,
            "confidence": 0.0
        }
    
    def _compute_twin_world_counterfactual(
        self,
        treatment: str,
        treatment_counterfactual: Any,
        outcome: str,
        observed_outcome: Any,
        observations: Dict[str, Any]
    ) -> Dict[str, Any]:
        """
        双世界モデルを使用して反事実を計算
        """
        # 事実世界を設定
        self.twin_world_model.setup_factual_world(observations)
        
        # 反事実世界を設定
        self.twin_world_model.setup_counterfactual_world(
            treatment=treatment,
            treatment_value=treatment_counterfactual
        )
        
        # 反事実結果を計算
        counterfactual_value = (
            self.twin_world_model.counterfactual_world.get(outcome)
        )
        
        return {
            "counterfactual_value": counterfactual_value,
            "probability": 1.0,
            "confidence": 0.7
        }
    
    def compute_partial_identification(
        self,
        treatment: str,
        treatment_counterfactual: Any,
        outcome: str,
        observed_outcome: Any,
        observations: Dict[str, Any]
    ) -> Dict[str, Any]:
        """
        部分識別境界を計算
        
        精密識別できない時、信用区間を返す
        """
        # ctf-calculusの部分識別理論を使用
        
        # 識別に必要な混淆変数を取得
        confounders = self._identify_confounders(treatment, outcome)
        
        # 境界を計算
        lower_bound = self._compute_lower_bound(
            treatment, treatment_counterfactual, outcome, confounders
        )
        upper_bound = self._compute_upper_bound(
            treatment, treatment_counterfactual, outcome, confounders
        )
        
        return {
            "identifiable": False,
            "partial": True,
            "lower_bound": lower_bound,
            "upper_bound": upper_bound,
            "confidence": 0.5,
            "method": "partial_identification"
        }
    
    def _identify_confounders(
        self,
        treatment: str,
        outcome: str
    ) -> List[str]:
        """混淆変数を識別"""
        # 後門基準を使用
        # フレームワーク実装
        return []
    
    def _compute_lower_bound(
        self,
        treatment: str,
        treatment_value: Any,
        outcome: str,
        confounders: List[str]
    ) -> float:
        """下界を計算"""
        return 0.0
    
    def _compute_upper_bound(
        self,
        treatment: str,
        treatment_value: Any,
        outcome: str,
        confounders: List[str]
    ) -> float:
        """上界を計算"""
        return 1.0
```

---

## §3. 反事実計算

### §3.1 反事実計算フロー

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                    反事実計算フロー                                   ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  ステップ1：事実世界を構築                                               ║
║  ───────────────────────────────                                    ║
║  - 観測された (X=x', Y=y')に基づいて構造方程式モデルを構築                      ║
║  - 外生変数Uの値を逆算                                              ║
║                                                                       ║
║  ステップ2：反事実干渉を設定                                            ║
║  ───────────────────────────────                                    ║
║  - 同じ構造方程式の下で、干渉do(X=x)を適用                              ║
║  - Uの値をそのまま保持（事実世界と同じ）                                 ║
║                                                                       ║
║  ステップ3：反事実結果を計算                                             ║
║  ───────────────────────────────                                    ║
║  - 反事実世界でY_xを解く                                             ║
║  - 反事実結果を取得                                                   ║
║                                                                       ║
║  ステップ4：反事実確率を計算                                            ║
║  ───────────────────────────────                                    ║
║  - 精密計算できない場合、P(Y_x = y | X=x', Y=y')を推定               ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### §3.2 反事実計算実装

```python
"""
反事実計算実装
"""

@dataclass
class CounterfactualResult:
    """反事実計算結果"""
    factual_value: Any           # 事実結果 Y(x')
    counterfactual_value: Any    # 反事実結果 Y(x)
    probability: float          # 反事実確率 P(Y_x = y | X=x', Y=y')
    causal_effect: float        # 個別処理効果 (ITE)
    assumptions: List[str]      # 識別仮定
    confidence: float           # 信心度


class CounterfactualEngine:
    """
    反事実計算エンジン
    """
    
    def __init__(self, sem: StructuralEquationModel):
        self.sem = sem
        self.twin_world_model = TwinWorldModel(sem)
    
    def compute_counterfactual(
        self,
        treatment: str,
        treatment_factual: Any,      # 実際に发生した処理
        treatment_counterfactual: Any, # 仮説の処理
        outcome: str,
        observed_outcome: Any,
        observations: Dict[str, Any]
    ) -> CounterfactualResult:
        """
        反事実結果を計算
        
        問題：「もし当初treatment_counterfactualを選択していたら、結果はどうだったか？」
        """
        
        # ═══════════════════════════════════════════════════
        # STEP 1: 事実世界を構築
        # ═══════════════════════════════════════════════════
        
        self.twin_world_model.setup_factual_world(observations)
        
        factual_value = self.twin_world_model.factual_world.get(outcome)
        
        # ═══════════════════════════════════════════════════
        # STEP 2: 反事実干渉を設定
        # ═══════════════════════════════════════════════════
        
        self.twin_world_model.setup_counterfactual_world(
            treatment=treatment,
            treatment_value=treatment_counterfactual
        )
        
        # ═══════════════════════════════════════════════════
        # STEP 3: 反事実結果を計算
        # ═══════════════════════════════════════════════════
        
        counterfactual_value = (
            self.twin_world_model.counterfactual_world.get(outcome)
        )
        
        # ═══════════════════════════════════════════════════
        # STEP 4: 個別処理効果を計算
        # ═══════════════════════════════════════════════════
        
        ite = counterfactual_value - factual_value if isinstance(
            factual_value, (int, float)
        ) else 0.0
        
        # ═══════════════════════════════════════════════════
        # STEP 5: 反事実確率を推定
        # ═══════════════════════════════════════════════════
        
        probability = self._estimate_counterfactual_probability(
            treatment,
            treatment_factual,
            treatment_counterfactual,
            outcome,
            observed_outcome
        )
        
        return CounterfactualResult(
            factual_value=factual_value,
            counterfactual_value=counterfactual_value,
            probability=probability,
            causal_effect=ite,
            assumptions=[
                "consistency",
                "conditional_exogeneity",
                "no_measurement_error"
            ],
            confidence=0.8
        )
    
    def _estimate_counterfactual_probability(
        self,
        treatment: str,
        treatment_factual: Any,
        treatment_counterfactual: Any,
        outcome: str,
        observed_outcome: Any
    ) -> float:
        """
        反事実確率を推定
        
        P(Y_x = y | X=x', Y=y')
        
        方法：
        1. 構造方程式が既知なら、精密計算
        2. 推定が必要な場合、感度分析を使用
        """
        # フレームワーク実装
        # 実装にはデータサポートが必要
        
        # 簡略化：確定性推定を返す
        return 1.0
    
    def compute_ite_distribution(
        self,
        treatment: str,
        outcome: str,
        individual_covariates: Dict[str, Any]
    ) -> List[float]:
        """
        個別処理効果分布を計算
        
        異質性分析に使用
        """
        ite_values = []
        
        # 各個人のITEを計算
        # フレームワーク実装
        
        return ite_values
```

---

## §4. 反事実因果推論

### §4.1 反事実因果推論タイプ

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                    反事実因果推論タイプ                                 ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  1. 責任帰因 (Attribution)                                          ║
║     ───────────────────────────────                                  ║
║     「誰が結果に責任があるか？」                                             ║
║     比較：事実結果 vs 反事実結果                                     ║
║                                                                       ║
║  2. 代理因果推論 (Proxy Causation)                                   ║
║     ───────────────────────────────                                  ║
║     「もしXが発生していなかったら、Yは发生したか？」                                 ║
║     反事実を使用して因果を定義                                       ║
║                                                                       ║
║  3. 結果解釈 (Outcome Explanation)                                  ║
║     ───────────────────────────────                                  ║
║     「なぜ結果はYだったか？」                                            ║
║     結果に至る因果チェーンを生成                                        ║
║                                                                       ║
║  4. 期待損失推定 (Expected Loss Estimation)                         ║
║     ───────────────────────────────                                  ║
║     「A而非Bを選択した場合の期待損失はいくら か？」                             ║
║     反事実を使用して機会コストを計算                                           ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### §4.2 反事実因果推論実装

```python
"""
反事実因果推論実装
"""

@dataclass
class AttributionResult:
    """責任帰因結果"""
    responsible_variable: str
    attribution_score: float
    counterfactual_comparison: Dict[str, Any]


@dataclass
class ExplanationResult:
    """結果解釈"""
    outcome: str
    causal_chain: List[str]
    explanation_confidence: float


class CounterfactualCausalInference:
    """
    反事実因果推論器
    """
    
    def __init__(self, causal_graph: "CausalGraph"):
        self.causal_graph = causal_graph
    
    def attribute_responsibility(
        self,
        outcome: str,
        observed_value: Any,
        potential_causes: List[str],
        observations: Dict[str, Any]
    ) -> AttributionResult:
        """
        責任帰因
        
        観測結果の主な原因を識別
        """
        best_cause = None
        best_score = -float('inf')
        
        for cause in potential_causes:
            # 反事実を計算：もし当該原因が発生していなかったら、結果はどうだったか？
            cf_result = self._compute_counterfactual_omission(
                cause, outcome, observations
            )
            
            # 帰因スコア：反事実結果と観測結果の差異
            attribution_score = self._compute_attribution_score(
                observed_value,
                cf_result.counterfactual_value
            )
            
            if attribution_score > best_score:
                best_score = attribution_score
                best_cause = cause
        
        return AttributionResult(
            responsible_variable=best_cause,
            attribution_score=best_score,
            counterfactual_comparison={}
        )
    
    def explain_outcome(
        self,
        outcome: str,
        outcome_value: Any,
        observations: Dict[str, Any]
    ) -> ExplanationResult:
        """
        結果解釈
        
        結果に至る因果チェーン解釈を生成
        """
        # 結果から根因までバックトラック
        causal_chain = self._backtrack_causal_chain(
            outcome, observations
        )
        
        return ExplanationResult(
            outcome=outcome_value,
            causal_chain=causal_chain,
            explanation_confidence=0.7
        )
    
    def estimate_expected_loss(
        self,
        action_taken: str,
        action_alternative: str,
        outcome: str,
        utility_function: Dict[Any, float],
        observations: Dict[str, Any]
    ) -> float:
        """
        期待損失を推定
        
        action_taken而非action_alternativeを選択した場合の機会コストを計算
        """
        # 事実結果の効用を計算
        factual_result = self._compute_factual_outcome(
            action_taken, outcome, observations
        )
        factual_utility = utility_function.get(factual_result, 0.0)
        
        # 反事実結果の効用を計算
        cf_result = self._compute_counterfactual_switch(
            action_taken, action_alternative, outcome, observations
        )
        cf_utility = utility_function.get(cf_result.counterfactual_value, 0.0)
        
        # 期待損失
        expected_loss = factual_utility - cf_utility
        
        return expected_loss
    
    def _compute_counterfactual_omission(
        self,
        variable: str,
        outcome: str,
        observations: Dict[str, Any]
    ) -> CounterfactualResult:
        """
        省略反事実を計算
        
        「もし当該変数が发生していなかったら、結果はどうだったか？」
        """
        # フレームワーク実装
        return CounterfactualResult(
            factual_value=0.0,
            counterfactual_value=0.0,
            probability=0.5,
            causal_effect=0.0,
            assumptions=[],
            confidence=0.5
        )
    
    def _compute_attribution_score(
        self,
        observed: Any,
        counterfactual: Any
    ) -> float:
        """帰因スコアを計算"""
        if isinstance(observed, (int, float)) and isinstance(
            counterfactual, (int, float)
        ):
            return abs(observed - counterfactual)
        return 0.0
    
    def _backtrack_causal_chain(
        self,
        outcome: str,
        observations: Dict[str, Any]
    ) -> List[str]:
        """因果チェーンをバックトラック"""
        chain = [outcome]
        
        # 簡略化実装
        current = outcome
        
        for _ in range(5):  # 最大5步
            parents = self.causal_graph.get_parents(current)
            if not parents:
                break
            
            # 最も関連深い親ノードを選択
            # フレームワーク実装
            break
        
        return chain
    
    def _compute_factual_outcome(
        self,
        action: str,
        outcome: str,
        observations: Dict[str, Any]
    ) -> Any:
        """事実結果を計算"""
        return observations.get(outcome, 0.0)
    
    def _compute_counterfactual_switch(
        self,
        action_taken: str,
        action_alternative: str,
        outcome: str,
        observations: Dict[str, Any]
    ) -> CounterfactualResult:
        """切り替え反事実を計算"""
        # フレームワーク実装
        return CounterfactualResult(
            factual_value=0.0,
            counterfactual_value=0.0,
            probability=0.5,
            causal_effect=0.0,
            assumptions=[],
            confidence=0.5
        )
```

---

## §5. 反事実意思決定応用

### §5.1 反事実意思決定サポート

```python
"""
反事実意思決定サポート
"""

class CounterfactualDecisionSupport:
    """
    反事実意思決定サポートシステム
    
    反事実推論を使用して高リスク意思決定をサポート
    """
    
    def __init__(self, causal_graph: "CausalGraph", sem: StructuralEquationModel):
        self.causal_graph = causal_graph
        self.sem = sem
        self.counterfactual_engine = CounterfactualEngine(sem)
        self.causal_inference = CounterfactualCausalInference(causal_graph)
    
    def analyze_decision(
        self,
        decision: str,
        outcome: str,
        alternatives: List[str],
        observations: Dict[str, Any],
        utility_function: Dict[Any, float]
    ) -> Dict[str, Any]:
        """
        意思決定を分析
        
        反事実推論を使用して意思決定の品質を評価
        """
        results = {
            "decision": decision,
            "analysis": {}
        }
        
        for alt in alternatives:
            if alt == decision:
                continue
            
            # 期待損失を計算
            expected_loss = self.causal_inference.estimate_expected_loss(
                decision, alt, outcome, utility_function, observations
            )
            
            # 責任帰因を計算
            attribution = self.causal_inference.attribute_responsibility(
                outcome, observations.get(outcome, 0.0),
                [decision, alt], observations
            )
            
            results["analysis"][alt] = {
                "expected_loss": expected_loss,
                "attribution": attribution.attribution_score,
                "recommendation": "avoid" if expected_loss > 0 else "acceptable"
            }
        
        return results
    
    def what_if_analysis(
        self,
        intervention: Dict[str, Any],
        outcome: str,
        baseline_observations: Dict[str, Any]
    ) -> Dict[str, Any]:
        """
        what-if分析
        
        「もし私たちがXに干渉したら、結果はどうなるか？」
        """
        treatment = list(intervention.keys())[0]
        treatment_value = intervention[treatment]
        
        # 反事実エンジンを使用して計算
        result = self.counterfactual_engine.compute_counterfactual(
            treatment=treatment,
            treatment_factual=baseline_observations.get(treatment, 0),
            treatment_counterfactual=treatment_value,
            outcome=outcome,
            observed_outcome=baseline_observations.get(outcome, 0),
            observations=baseline_observations
        )
        
        return {
            "intervention": intervention,
            "factual": result.factual_value,
            "counterfactual": result.counterfactual_value,
            "causal_effect": result.causal_effect,
            "confidence": result.confidence
        }
```

---

## §6. LOGIC_ENGINEとの統合

### §6.1 統合インターフェース

```python
"""
反事実推論とロジックエンジン統合
"""

def apply_counterfactual_to_decision(
    decision_input: Dict,
    context: "DecisionContext"
) -> "DecisionOutput":
    """
    反事実推論を意思決定フローに適用
    
    「もし...だったらどうなるか？」という类型の問題に答える必要がある時に起動
    """
    from CAUSAL_INFERENCE import CausalInferenceEngine
    
    # 1. 意思決定関連変数を抽出
    decision_var = decision_input.get("decision")
    outcome_var = decision_input.get("outcome")
    alternatives = decision_input.get("alternatives", [])
    
    # 2. 因果グラフを取得
    graph = BuildOrRetrieveCausalGraph(decision_input, context)
    
    # 3. 反事実エンジンを初期化
    # 構造方程式モデルが必要
    sem = StructuralEquationModel(
        exogenous_vars=set(),
        endogenous_vars=set(),
        structural_equations={}
    )
    
    cf_engine = CounterfactualEngine(sem)
    cf_inference = CounterfactualCausalInference(graph)
    
    # 4. 反事実分析を実行
    results = {}
    
    for alt in alternatives:
        cf_result = cf_engine.compute_counterfactual(
            treatment=decision_var,
            treatment_factual=decision_input.get("current_value", ""),
            treatment_counterfactual=alt,
            outcome=outcome_var,
            observed_outcome=decision_input.get("observed_outcome", 0),
            observations=decision_input.get("observations", {})
        )
        
        results[alt] = {
            "counterfactual_value": cf_result.counterfactual_value,
            "causal_effect": cf_result.causal_effect,
            "confidence": cf_result.confidence
        }
    
    # 5. 意思決定提案を生成
    best_alternative = max(
        results.keys(),
        key=lambda a: results[a]["causal_effect"]
    )
    
    return {
        "recommended_action": best_alternative,
        "analysis": results,
        "method": "counterfactual_reasoning"
    }
```

---

## §7. 依存と制約

### §7.1 モジュール依存

| 依存モジュール | 説明 | 参照 |
| --- | --- | --- |
| LOGIC_ENGINE.md | 親モジュール | 呼び出しエントリ |
| CAUSAL_INFERENCE.md | 因果推論エンジン | do-calculus |
| ABDUCTIVE_REASONING.md | 帰納推論 | 異常解釈 |
| CONSTRAINTS.md | 社会権限レベル | SA-L権限校验 |
| FORMAL_VERIFIER.md | 形式化検証 | 論理閉包検証 |

### §7.2 制約条件

| 制約タイプ | 説明 | 境界 |
| --- | --- | :--- |
| 構造方程式 | 完全なSEMが必要 | フレームワーク実装 |
| 外生変数 | 推論可能または観測可能が必要 | 制限あり |
| 信心度閾値 | 信心度 < 0.3 は不確定を标注 | confidence ≧ 0.3 |
| 反事実深度 | 最大バックトラック深度 | ≦ 5步 |

---

## §8. バージョンと進化

| バージョン | 日付 | 変更要約 |
| --- | :--- | :--- |
| v2.2 | 2026-03 | 初期バージョン、LOGIC_ENGINE.md §2.3に対応 |
| v2.3 | 2026-03 | ctf-calculusを統合して反事実推論を強化 |

**進化制約：** 本モジュールの修改はNoieLogicAGENTS.mdの不変コア公理に反してはならない。任意の進化提案はEVOLUTION_LOG.mdに記録されなければならない。

---

*Counterfactual Reasoning Framework v2.2 — 双世界モデルと反事実因果推論*
*第3層因果推論のコア能力を実装*
