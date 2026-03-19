# ABDUCTIVE_REASONING.md

## 帰納推論モジュール (Abductive Reasoning Module v2.2)

**定義：** 本モジュールはNoieLogicAGENTSロジックエンジンのコアサブモジュールであり、帰納推論（Abductive Reasoning）フレームワークを実装し、異常検出、候補原因生成と最短説明選択專門担当。

**システム定位：** LOGIC_ENGINE.mdのL3詳細モジュールとして、第3層認知能力專門担当——観測結果が既存因果グラフで説明できない時、新しい因果仮説を生成する。

**依存モジュール：** LOGIC_ENGINE.md、CAUSAL_INFERENCE.md、CONSTRAINTS.md、FORMAL_VERIFIER.md

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

## §1. 帰納推論理論基盤

### §1.1 帰納推論定義

帰納推論（Abduction）はCharles Sanders Peirceが提唱した第三の推論形式で、演繹（Deduction）と帰納（Induction）と並列する。

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                    三種類の推論形式比較                                 ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  【演繹推論 (Deduction)】                                            ║
║  ─────────────────────────────────────                              ║
║  前提：すべてのAはBである、xはAである                                          ║
║  結論：xはBである                                                       ║
║  性質：保真推論（前提が真なら、結論も必ず真）                         ║
║                                                                       ║
║  【帰納推論 (Induction)】                                            ║
║  ─────────────────────────────────────                              ║
║  前提：x₁はAかつB、x₂はAかつB、...                            ║
║  結論：すべてのAはBである                                                  ║
║  性質：確率推論（結論は概然的）                                       ║
║                                                                       ║
║  【帰納推論 (Abduction)】                                            ║
║  ─────────────────────────────────────                              ║
║  前提：Bが観測された、ルール A → B                                         ║
║  結論：xはAかもしれない                                                   ║
║  性質：最良説明推論（結論は仮説）                                  ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### §1.2 帰納推論が因果フレームワークにおける役割

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                    三層因果推論と帰納推論                           ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  Layer 3 ────────── 反事実 (Counterfactual) ────────── 最高レベル    ║
║  「もし当初違う選択をしていたら、結果はどうだったか？」                               ║
║                                                                       ║
║        ═══════════════════════════════════════                       ║
║                                                                       ║
║  Layer 2 ────────── 干渉 (Intervention) ────────── 中間レベル         ║
║  「もし私がXを強制的に変えたら、Yはどう変わる？」                                 ║
║                                                                       ║
║        ═══════════════════════════════════════                       ║
║                                                                       ║
║  Layer 1 ────────── 関連 (Association) ────────── 基底レベル          ║
║  「Xが观测された時、Yの確率は？」                             ║
║                                                                       ║
║        ═══════════════════════════════════════                       ║
║                                                                       ║
║  【帰納推論の位置】                                                    ║
║  ─────────────────────────────────────                              ║
║  帰納推論は「因果学習」のコアである：                                     ║
║  観測と因果グラフ予測が一致しない時、新しい因果仮説を生成                       ║
║  これはデータから因果構造を発見するコア能力                                 ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

---

## §2. 異常検出

### §2.1 異常定義とタイプ

```python
"""
異常検出モジュール
"""

from typing import Dict, List, Optional, Set, Tuple
from dataclasses import dataclass
from enum import Enum
import math

class AnomalyType(Enum):
    """異常タイプ"""
    RESIDUAL = "residual"           # 残差異常
    STRUCTURAL = "structural"        # 構造異常
    DISTRIBUTIONAL = "distributional" # 分布異常
    TEMPORAL = "temporal"            # 時系列異常
    CAUSAL = "causal"               # 因果異常

@dataclass
class Anomaly:
    """異常構造"""
    anomaly_type: AnomalyType
    observed_value: float
    predicted_value: float
    residual: float
    severity: float  # 0-1
    timestamp: Optional[str] = None
    context: Optional[Dict] = None

@dataclass
class AnomalyDetectionResult:
    """異常検出結果"""
    is_anomaly: bool
    anomaly: Optional[Anomaly]
    detection_method: str
    confidence: float
    threshold_used: float


class AnomalyDetector:
    """
    異常検出器
    """
    
    def __init__(
        self,
        residual_threshold: float = 2.0,
        severity_threshold: float = 0.5
    ):
        self.residual_threshold = residual_threshold
        self.severity_threshold = severity_threshold
    
    def detect_residual_anomaly(
        self,
        observed: float,
        predicted: float,
        std_error: float = 1.0
    ) -> AnomalyDetectionResult:
        """
        残差異常を検出
        
        標準化残差を使用：
        z = (observed - predicted) / std_error
        
        もし|z| > thresholdなら、異常である
        """
        residual = observed - predicted
        z_score = abs(residual / std_error) if std_error > 0 else float('inf')
        
        is_anomaly = z_score > self.residual_threshold
        
        # 重大度を計算
        severity = min(z_score / self.residual_threshold, 1.0)
        
        anomaly = None
        if is_anomaly:
            anomaly = Anomaly(
                anomaly_type=AnomalyType.RESIDUAL,
                observed_value=observed,
                predicted_value=predicted,
                residual=residual,
                severity=severity
            )
        
        return AnomalyDetectionResult(
            is_anomaly=is_anomaly,
            anomaly=anomaly,
            detection_method="residual_zscore",
            confidence=min(severity, 0.95),
            threshold_used=self.residual_threshold
        )
    
    def detect_distributional_anomaly(
        self,
        value: float,
        distribution_params: Dict
    ) -> AnomalyDetectionResult:
        """
        分布異常を検出
        
        分布仮定（例：正規分布）を使用して尾部確率を計算
        """
        # 簡略化実装：Z-scoreを使用
        mean = distribution_params.get("mean", 0)
        std = distribution_params.get("std", 1)
        
        return self.detect_residual_anomaly(value, mean, std)
    
    def detect_temporal_anomaly(
        self,
        time_series: List[float],
        window_size: int = 10
    ) -> List[AnomalyDetectionResult]:
        """
        時系列異常を検出
        
        滑动ウィンドウを使用してトレンドから逸脱する点を検出
        """
        anomalies = []
        
        for i in range(window_size, len(time_series)):
            # ウィンドウの平均値を計算
            window = time_series[i-window_size:i]
            predicted = sum(window) / window_size
            
            # 異常を検出
            result = self.detect_residual_anomaly(
                time_series[i],
                predicted
            )
            
            if result.is_anomaly and result.anomaly:
                result.anomaly.timestamp = f"t={i}"
            
            anomalies.append(result)
        
        return anomalies
```

### §2.2 因果異常検出

```python
"""
因果異常検出
"""

class CausalAnomalyDetector:
    """
    因果異常検出器
    
    観測結果が因果グラフの予測から逸脱するかを検出
    """
    
    def __init__(self, causal_engine: "CausalInferenceEngine"):
        self.causal_engine = causal_engine
        self.anomaly_detector = AnomalyDetector()
    
    def detect_causal_anomaly(
        self,
        graph: "CausalGraph",
        observation: Dict[str, float],
        conditions: Dict[str, float]
    ) -> AnomalyDetectionResult:
        """
        因果異常を検出
        
        ステップ：
        1. 因果グラフを使用して結果を予測
        2. 予測と観測を比較
        3. 逸脱が閾値を超えていれば異常として标注
        """
        # 観測から目標変数を抽出
        target_var = list(observation.keys())[0]
        
        # グラフのノードかチェック
        if target_var not in graph.nodes:
            return AnomalyDetectionResult(
                is_anomaly=False,
                anomaly=None,
                detection_method="causal_not_applicable",
                confidence=0.0,
                threshold_used=0.0
            )
        
        # 因果エンジンを使用して予測
        treatment = list(conditions.keys())[0] if conditions else None
        
        if treatment:
            effect_result = self.causal_engine.estimate_causal_effect(
                X=treatment,
                Y=target_var
            )
            predicted = effect_result.ate
        else:
            # 干渉なし、基本予測を使用
            predicted = 0.0
        
        observed = observation[target_var]
        
        # 異常を検出
        return self.anomaly_detector.detect_residual_anomaly(
            observed,
            predicted
        )
```

---

## §3. 候補原因生成

### §3.1 原因空間定義

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                    候補原因空間                                      ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  異常が観測された時、候補原因の空間を搜索する定義：                           ║
║                                                                       ║
║  C(異常) = { c | 異常 ∈ Consequence(c) }                          ║
║                                                                       ║
║  ただし：                                                              ║
║    - cは候補原因                                                   ║
║    - Consequence(c)は原因c可能导致的后果の集合                     ║
║                                                                       ║
║  候補原因タイプ：                                                      ║
║    1. 直接原因：異常ノードに直接指す辺の起点                          ║
║    2. 間接原因：因果パスを通じて異常に影響するノード                         ║
║    3. 根本原因：他の原因のない原因（入辺がないか、外生変数からの入辺）       ║
║    4. 混淆原因：後門パスを通じて異常に影響するノード                        ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### §3.2 候補原因検索アルゴリズム

```python
"""
候補原因生成
"""

@dataclass
class CandidateCause:
    """候補原因構造"""
    variable: str
    cause_type: str  # direct, indirect, root, confounding
    causal_path: List[str]  # 原因から異常までのパス
    probability: float  # 事後確率
    complexity: float  # コルモゴロフ複雑度


class CandidateCauseGenerator:
    """
    候補原因生成器
    """
    
    def __init__(self, graph: "CausalGraph"):
        self.graph = graph
    
    def generate_candidates(
        self,
        anomaly_variable: str,
        max_depth: int = 3
    ) -> List[CandidateCause]:
        """
        候補原因リストを生成
        """
        candidates = []
        
        # 1. 直接原因を検索
        direct_causes = self._find_direct_causes(anomaly_variable)
        for cause in direct_causes:
            candidates.append(CandidateCause(
                variable=cause,
                cause_type="direct",
                causal_path=[cause, anomaly_variable],
                probability=0.8,
                complexity=self._estimate_complexity(cause)
            ))
        
        # 2. 間接原因を検索
        indirect_causes = self._find_indirect_causes(
            anomaly_variable, max_depth
        )
        for cause in indirect_causes:
            path = self._find_causal_path(cause, anomaly_variable)
            candidates.append(CandidateCause(
                variable=cause,
                cause_type="indirect",
                causal_path=path,
                probability=0.5,
                complexity=self._estimate_complexity(cause)
            ))
        
        # 3. 根本原因を識別
        root_causes = self._find_root_causes(anomaly_variable)
        for cause in root_causes:
            path = self._find_causal_path(cause, anomaly_variable)
            candidates.append(CandidateCause(
                variable=cause,
                cause_type="root",
                causal_path=path,
                probability=0.9,
                complexity=self._estimate_complexity(cause)
            ))
        
        # 4. 混淆原因を検索
        confounding_causes = self._find_confounding_causes(anomaly_variable)
        for cause in confounding_causes:
            path = self._find_backdoor_path(cause, anomaly_variable)
            candidates.append(CandidateCause(
                variable=cause,
                cause_type="confounding",
                causal_path=path,
                probability=0.4,
                complexity=self._estimate_complexity(cause)
            ))
        
        return candidates
    
    def _find_direct_causes(self, target: str) -> List[str]:
        """直接原因を検索"""
        return list(self.graph.get_parents(target))
    
    def _find_indirect_causes(
        self, 
        target: str, 
        max_depth: int
    ) -> List[str]:
        """間接原因を検索"""
        indirect = []
        visited = {target}
        
        def dfs(node: str, depth: int):
            if depth >= max_depth:
                return
            
            for parent in self.graph.get_parents(node):
                if parent not in visited:
                    visited.add(parent)
                    # 直接原因を除外
                    if parent not in self.graph.get_parents(target):
                        indirect.append(parent)
                    dfs(parent, depth + 1)
        
        dfs(target, 0)
        return indirect
    
    def _find_root_causes(self, target: str) -> List[str]:
        """根本原因を検索"""
        root_causes = []
        
        # targetへのすべてのパスを遍历
        for node in self.graph.nodes:
            if node == target:
                continue
            
            # 根ノードかチェック（親がないか、親が外生変数）
            parents = self.graph.get_parents(node)
            if len(parents) == 0:
                # targetへのパスがあるかチェック
                if self._has_path_to(node, target):
                    root_causes.append(node)
        
        return root_causes
    
    def _find_confounding_causes(self, target: str) -> List[str]:
        """混淆原因を検索"""
        # 混淆原因：後門パスを通じてtargetに影響するノード
        # 簡略化実装
        return []
    
    def _has_path_to(self, source: str, target: str) -> bool:
        """sourceからtargetへのパスがあるかをチェック"""
        visited = set()
        queue = [source]
        
        while queue:
            node = queue.pop(0)
            if node == target:
                return True
            
            if node in visited:
                continue
            
            visited.add(node)
            
            for child in self.graph.get_children(node):
                if child not in visited:
                    queue.append(child)
        
        return False
    
    def _find_causal_path(
        self, 
        source: str, 
        target: str
    ) -> List[str]:
        """sourceからtargetへの因果パスを検索"""
        path = []
        visited = set()
        
        def dfs(node: str, current_path: List[str]):
            if node == target:
                path.extend(current_path + [node])
                return True
            
            visited.add(node)
            
            for child in self.graph.get_children(node):
                if child not in visited:
                    if dfs(child, current_path + [node]):
                        return True
            
            return False
        
        dfs(source, [])
        return path
    
    def _find_backdoor_path(
        self, 
        source: str, 
        target: str
    ) -> List[str]:
        """後門パスを検索"""
        # 簡略化実装
        return []
    
    def _estimate_complexity(self, variable: str) -> float:
        """
        変数のコルモゴロフ複雑度を推定
        
        簡略化：変数名の長さとグラフでの次数をプロキシとして使用
        """
        name_complexity = len(variable) / 10.0
        
        connections = (
            len(self.graph.get_parents(variable)) +
            len(self.graph.get_children(variable))
        )
        structural_complexity = connections / 10.0
        
        return min(name_complexity + structural_complexity, 1.0)
```

---

## §4. 最短説明選択

### §4.1 コルモゴロフ複雑度

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                    最短記述優先原則                                   ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  【コルモゴロフ複雑度定義】                                          ║
║                                                                       ║
║  K(x) = xを生成する最短プログラムの長さ                                     ║
║                                                                       ║
║  因果推論において：                                                       ║
║    K(説明) ∝ 説明の記述長さ + 因果パス長さ                          ║
║                                                                       ║
║  【最短説明選択基準】                                                ║
║                                                                       ║
║  K(説明)가最小の候補原因を選択：                                       ║
║    Best = argmin_c K(c | 異常)                                      ║
║                                                                       ║
║  これは「オルカムの剃刀」原則を体現している：                                        ║
║    「もし必要なければ、実体を増やすな」                                           ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### §4.2 最良説明選択アルゴリズム

```python
"""
最短説明選択
"""

class BestExplanationSelector:
    """
    最短説明選択器
    
    コルモゴロフ複雑度を使用して最良説明を選択
    """
    
    def __init__(self):
        self.kolmogorov_constant = 1.0  # 正規化定数
    
    def select_best_explanation(
        self,
        candidates: List[CandidateCause],
        observation: Dict
    ) -> CandidateCause:
        """
        最良説明を選択
        
        選択基準：
        P(原因|異常) ∝ P(異常|原因) × P(原因) / K(原因)
        
        ただし：
        - P(異常|原因)：原因の異常への説明力
        - P(原因)：原因の事前確率
        - K(原因)：原因のコルモゴロフ複雑度
        """
        scored_candidates = []
        
        for candidate in candidates:
            # 事後スコアを計算
            posterior = self._compute_posterior(candidate, observation)
            
            # 複雑度を考慮
            complexity_penalty = candidate.complexity * self.kolmogorov_constant
            
            # 最終スコア
            score = posterior - complexity_penalty
            
            scored_candidates.append((candidate, score))
        
        # スコアでソート
        scored_candidates.sort(key=lambda x: x[1], reverse=True)
        
        # 最高スコアを返す
        return scored_candidates[0][0] if scored_candidates else None
    
    def _compute_posterior(
        self,
        candidate: CandidateCause,
        observation: Dict
    ) -> float:
        """
        事後確率を計算
        
        P(原因|異常) ∝ P(異常|原因) × P(原因)
        """
        # 事前確率（原因タイプに基づく）
        prior = self._compute_prior(candidate)
        
        # 尤度（因果パスに基づく）
        likelihood = self._compute_likelihood(candidate, observation)
        
        # 事後
        posterior = prior * likelihood
        
        return posterior
    
    def _compute_prior(self, candidate: CandidateCause) -> float:
        """事前確率を計算"""
        # 根本原因が最高優先級
        type_prior = {
            "root": 0.9,
            "direct": 0.7,
            "indirect": 0.4,
            "confounding": 0.3
        }
        
        return type_prior.get(candidate.cause_type, 0.5)
    
    def _compute_likelihood(
        self,
        candidate: CandidateCause,
        observation: Dict
    ) -> float:
        """
        尤度を計算
        
        因果パスの強度に基于
        """
        # 簡略化：パスが短いほど尤度が高い
        path_length = len(candidate.causal_path)
        
        if path_length == 0:
            return 0.0
        
        return 1.0 / path_length
    
    def rank_explanations(
        self,
        candidates: List[CandidateCause],
        observation: Dict
    ) -> List[Tuple[CandidateCause, float]]:
        """
        すべての候補原因をソート
        
        返回：(原因, スコア)の順序付きリスト
        """
        scored = []
        
        for candidate in candidates:
            posterior = self._compute_posterior(candidate, observation)
            complexity_penalty = candidate.complexity * self.kolmogorov_constant
            score = posterior - complexity_penalty
            
            scored.append((candidate, score))
        
        return sorted(scored, key=lambda x: x[1], reverse=True)
```

---

## §5. 帰納推論フロー

### §5.1 完全帰納推論フロー

```python
"""
帰納推論完全フロー
"""

@dataclass
class AbductiveResult:
    """帰納推論結果"""
    status: str  # "anomaly_detected", "no_anomaly", "inconclusive"
    anomaly: Optional[Anomaly]
    best_explanation: Optional[CandidateCause]
    all_candidates: List[CandidateCause]
    ranked_explanations: List[Tuple[CandidateCause, float]]
    graph_update_proposed: bool
    confidence: float


class AbductiveReasoner:
    """
    帰納推論器
    """
    
    def __init__(
        self,
        causal_engine: "CausalInferenceEngine",
        graph: "CausalGraph"
    ):
        self.causal_engine = causal_engine
        self.graph = graph
        self.anomaly_detector = AnomalyDetector()
        self.cause_generator = CandidateCauseGenerator(graph)
        self.explanation_selector = BestExplanationSelector()
    
    def infer(
        self,
        observation: Dict[str, float],
        conditions: Optional[Dict[str, float]] = None
    ) -> AbductiveResult:
        """
        完全な帰納推論フローを実行
        
        ステップ：
        1. 異常検出
        2. 異常が検出されたら、候補原因を生成
        3. 最良説明を選択
        4. 信心度が十分なら因果グラフ更新を提案
        """
        # ═══════════════════════════════════════════════════
        # STEP 1: 異常検出
        # ═══════════════════════════════════════════════════
        
        target_var = list(observation.keys())[0]
        observed_value = observation[target_var]
        
        # 因果エンジンを使用して予測
        if conditions:
            treatment = list(conditions.keys())[0]
            effect_result = self.causal_engine.estimate_causal_effect(
                X=treatment,
                Y=target_var
            )
            predicted = effect_result.ate
        else:
            predicted = 0.0
        
        anomaly_result = self.anomaly_detector.detect_residual_anomaly(
            observed_value,
            predicted
        )
        
        # ═══════════════════════════════════════════════════
        # 異常がなければ、正常を返す
        # ═══════════════════════════════════════════════════
        
        if not anomaly_result.is_anomaly:
            return AbductiveResult(
                status="no_anomaly",
                anomaly=None,
                best_explanation=None,
                all_candidates=[],
                ranked_explanations=[],
                graph_update_proposed=False,
                confidence=anomaly_result.confidence
            )
        
        # ═══════════════════════════════════════════════════
        # STEP 2: 候補原因を生成
        # ═══════════════════════════════════════════════════
        
        candidates = self.cause_generator.generate_candidates(target_var)
        
        if not candidates:
            return AbductiveResult(
                status="inconclusive",
                anomaly=anomaly_result.anomaly,
                best_explanation=None,
                all_candidates=[],
                ranked_explanations=[],
                graph_update_proposed=False,
                confidence=0.3
            )
        
        # ═══════════════════════════════════════════════════
        # STEP 3: 最良説明を選択
        # ═══════════════════════════════════════════════════
        
        ranked = self.explanation_selector.rank_explanations(
            candidates, observation
        )
        
        best = ranked[0][0] if ranked else None
        
        # ═══════════════════════════════════════════════════
        # STEP 4: 信心度が十分なら因果グラフ更新を提案
        # ═══════════════════════════════════════════════════
        
        UPDATE_THRESHOLD = 0.8
        
        graph_update_proposed = False
        if best and best.probability > UPDATE_THRESHOLD:
            graph_update_proposed = True
            # 監査に記録
            self._log_graph_update_proposal(best, observation)
        
        return AbductiveResult(
            status="anomaly_detected",
            anomaly=anomaly_result.anomaly,
            best_explanation=best,
            all_candidates=candidates,
            ranked_explanations=ranked[:5],  # 上位5名
            graph_update_proposed=graph_update_proposed,
            confidence=best.probability if best else 0.3
        )
    
    def _log_graph_update_proposal(
        self,
        cause: CandidateCause,
        observation: Dict
    ):
        """因果グラフ更新提案を記録"""
        # 実際の実装：AUDIT_TRAILに書き込み
        pass
```

---

## §6. LOGIC_ENGINEとの統合

### §6.1 統合インターフェース

```python
"""
帰納推論とロジックエンジン統合
"""

def apply_abductive_reasoning_to_observation(
    observation: Dict,
    causal_graph: "CausalGraph",
    context: "DecisionContext"
) -> "AbductiveResult":
    """
    帰納推論を観測分析に適用
    
    意思決定エンジンが異常を検出した時に起動：
    1. 観測結果を分析
    2. 候補因果解釈を生成
    3. 最良解釈を選択
    4. 因果グラフ更新を提案
    """
    from CAUSAL_INFERENCE import CausalInferenceEngine
    
    # 因果エンジンを初期化
    engine = CausalInferenceEngine(causal_graph)
    
    # 帰納推論を実行
    reasoner = AbductiveReasoner(engine, causal_graph)
    result = reasoner.infer(observation)
    
    # 監査に記録
    if result.status == "anomaly_detected":
        # 異常検出を記録
        pass
    
    return result
```

---

## §7. 依存と制約

### §7.1 モジュール依存

| 依存モジュール | 説明 | 参照 |
| --- | --- | --- |
| LOGIC_ENGINE.md | 親モジュール | 呼び出しエントリ |
| CAUSAL_INFERENCE.md | 因果推論エンジン | 異常予測 |
| CONSTRAINTS.md | 社会権限レベル | SA-L権限校验 |
| FORMAL_VERIFIER.md | 形式化検証 | 仮説検証 |

### §7.2 制約条件

| 制約タイプ | 説明 | 境界 |
| --- | --- | :--- |
| 異常閾値 | 残差 > 2σ を異常として标注 | z > 2.0 |
| 更新閾値 | 信心度 > 0.8 で更新を提案 | confidence ≧ 0.8 |
| 原因深度 | 搜索深度 ≦ 3 | max_depth ≦ 3 |
| 候補数 | 最大10個の候補を保持 | top 10 |

---

## §8. バージョンと進化

| バージョン | 日付 | 変更要約 |
| --- | :--- | :--- |
| v2.2 | 2026-03 | 初期バージョン、LOGIC_ENGINE.md §3に対応 |

**進化制約：** 本モジュールの修改はNoieLogicAGENTS.mdの不変コア公理に反してはならない。任意の進化提案はEVOLUTION_LOG.mdに記録されなければならない。

---

*Abductive Reasoning Module v2.2 — 異常検出、原因生成と最短説明選択*
*因果学習と仮説生成のコア能力を実装*
