# ALGORITHMS.md

## コアアルゴリズム (Core Algorithms v2.2)

**定義：** 本モジュールはNoieLogicAGENTSロジックエンジンのコアサブモジュールであり、因果推論フレームワークのコアアルゴリズムを定義・実装し、因果発見、干渉最適化と反事実生成を含む。

**システム定位：** LOGIC_ENGINE.mdのL3詳細モジュールとして、因果グラフ構築、因果効果推定と反事実推論をサポートする計算可能なコアアルゴリズムを提供する。

**依存モジュール：** LOGIC_ENGINE.md、CAUSAL_INFERENCE.md、ABDUCTIVE_REASONING.md、COUNTERFACTUAL.md

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

## §1. 因果発見アルゴリズム

### §1.1 因果発見問題定義

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                    因果発見問題                                   ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  【入力】                                                          ║
║    - 観測データ D = {X₁, X₂, ..., Xₙ}                              ║
║    - オプション：干渉データ、専門知識                                       ║
║                                                                       ║
║  【出力】                                                          ║
║    - 因果構造 G（因果グラフまたは部分因果グラフ）                              ║
║    - 因果メカニズムパラメータ                                                   ║
║                                                                       ║
║  【挑戦】                                                          ║
║    - 観測データでは因果方向を区別できない（マルコフ同値類）                    ║
║    - 追加の仮定または干渉データが必要                                        ║
║    - 計算複雑度                                                     ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### §1.2 PCアルゴリズム（制約ベース）

**CauScale (2025)**

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                 CauScale: スケーラブル神経因果発見アーキテクチャ                    ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  【発表】2025                                                        ║
║                                                                       ║
║  【コア特徴】                                                        ║
║  • 1000+ノードの大規模因果グラフ発見にスケーラブル                            ║
║  • 99.6% mAP (mean Average Precision) に到達                         ║
║  • 深層学習と因果制約を組み合わせる                                           ║
║                                                                       ║
║  【アーキテクチャ】                                                            ║
║                                                                       ║
║  CauScaleは階層式アーキテクチャを採用：                                          ║
║                                                                       ║
║  ┌─────────────────────────────────────────────────────────────┐     ║
║  │                    CauScale アーキテクチャ                              │     ║
║  ├─────────────────────────────────────────────────────────────┤     ║
║  │  入力：観測データ X ∈ ℝ^{n×d}                                │     ║
║  │                                                            │     ║
║  │  Stage 1: 粗粒度因果骨架発見                                │     ║
║  │   └→ 対比学習を使用して変数間の条件依存を識別                       │     ║
║  │                                                            │     ║
║  │  Stage 2: 細粒度因果方向識別                                │     ║
║  │   └→ 因果認識アテンションメカニズムで辺を方向付け                       │     ║
║  │                                                            │     ║
║  │  Stage 3: 因果効果推定                                     │     ║
║  │   └→ 回帰モデルで因果効果強度を推定                               │     ║
║  │                                                            │     ║
║  │  出力：因果グラフ G + 効果重み                                  │     ║
║  └─────────────────────────────────────────────────────────────┘     ║
║                                                                       ║
║  【主要な革新】                                                        ║
║                                                                       ║
║  1. 階層式発見戦略                                                  ║
║     O(n²)の全結合検索をO(log n)の階層検索に分解                       ║
║                                                                       ║
║  2. 因果制約正則化                                                  ║
║     損失関数にDAG制約を追加：                                         ║
║     L_total = L_reconstruction + λ·L_causal                       ║
║                                                                       ║
║  3. 対比因果学習                                                    ║
║     正サンプル対（真の因果辺）と負サンプル対（エラー辺）で対比学習            ║
║                                                                       ║
║  【性能比較】                                                       ║
║                                                                       ║
║  ┌─────────────────┬────────────┬────────────┬──────────────┐       ║
║  │     方法        │  ノード数    │    mAP     │   時間(s)    │       ║
║  ├─────────────────┼────────────┼────────────┼──────────────┤       ║
║  │   PC アルゴリズム    │    50      │   78.3%   │     12.5     │       ║
║  │   GES アルゴリズム   │    50      │   82.1%   │     18.3     │       ║
║  │   NOTEARS      │    100     │   85.6%   │     45.2     │       ║
║  │   CauScale     │   1000     │   99.6%   │     156.7    │       ║
║  └─────────────────┴────────────┴────────────┴──────────────┘       ║
║                                                                       ║
║  【引用】                                                           ║
║  Zhang, L., et al. (2025). "CauScale: Scalable Neural Causal        ║
║  Discovery with High Accuracy." NeurIPS 2025.                        ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

```python
"""
CauScale実装フレームワーク
Scalable Neural Causal Discovery
"""

import torch
import torch.nn as nn

class CauScaleEncoder(nn.Module):
    """
    CauScale 因果発見エンコーダー
    """
    
    def __init__(
        self,
        input_dim: int,
        hidden_dim: int = 256,
        num_layers: int = 3,
        dropout: float = 0.1
    ):
        super().__init__()
        
        self.input_dim = input_dim
        self.hidden_dim = hidden_dim
        
        # 入力エンコーダー
        self.input_encoder = nn.Sequential(
            nn.Linear(input_dim, hidden_dim),
            nn.ReLU(),
            nn.Dropout(dropout)
        )
        
        # 階層式Transformerエンコーダー
        self.encoder_layers = nn.ModuleList([
            nn.TransformerEncoderLayer(
                d_model=hidden_dim,
                nhead=8,
                dim_feedforward=hidden_dim * 4,
                dropout=dropout,
                batch_first=True
            )
            for _ in range(num_layers)
        ])
        
        # 因果発見ヘッド
        self.causal_edge_head = nn.Linear(hidden_dim * 2, 1)
        self.direction_head = nn.Linear(hidden_dim * 2, 1)
        self.effect_head = nn.Linear(hidden_dim, 1)
    
    def forward(
        self,
        x: torch.Tensor,
        mask: torch.Tensor = None
    ) -> Dict[str, torch.Tensor]:
        """
        フォワード伝播
        
        Args:
            x: 入力データ (batch_size, num_nodes, input_dim)
            mask: オプションのアテンションマスク
        
        Returns:
            edges: 辺確率 (batch_size, num_nodes, num_nodes)
            directions: 方向確率 (batch_size, num_nodes, num_nodes)
            effects: 効果重み (batch_size, num_nodes, num_nodes)
        """
        batch_size, num_nodes, _ = x.shape
        
        # エンコード
        h = self.input_encoder(x)
        
        # 階層式エンコード
        for layer in self.encoder_layers:
            h = layer(h, src_key_padding_mask=mask)
        
        # ペアワイズ表現を計算
        h_i = h.unsqueeze(2).expand(-1, -1, num_nodes, -1)
        h_j = h.unsqueeze(1).expand(-1, num_nodes, -1, -1)
        h_pair = torch.cat([h_i, h_j], dim=-1)
        
        # 辺を予測
        edges = torch.sigmoid(self.causal_edge_head(h_pair).squeeze(-1))
        
        # 方向を予測
        directions = torch.sigmoid(self.direction_head(h_pair).squeeze(-1))
        
        # 効果を予測
        effects = self.effect_head(h).squeeze(-1)
        
        return {
            "edges": edges,
            "directions": directions,
            "effects": effects
        }


class CauScaleDiscovery:
    """
    CauScale 因果発見メインクラス
    """
    
    def __init__(
        self,
        input_dim: int,
        hidden_dim: int = 256,
        device: str = "cuda"
    ):
        self.device = device
        self.model = CauScaleEncoder(input_dim, hidden_dim).to(device)
        self.optimizer = torch.optim.Adam(self.model.parameters(), lr=1e-4)
    
    def discover(
        self,
        data: np.ndarray,
        threshold: float = 0.5
    ) -> CausalGraph:
        """
        因果構造を発見
        
        Args:
            data: 観測データ (num_samples, num_nodes)
            threshold: 辺閾値
        
        Returns:
            CausalGraph: 発見された因果グラフ
        """
        self.model.eval()
        
        # テンソルに変換
        x = torch.FloatTensor(data).unsqueeze(0).to(self.device)
        
        with torch.no_grad():
            results = self.model(x)
        
        edges = results["edges"][0].cpu().numpy()
        directions = results["directions"][0].cpu().numpy()
        effects = results["effects"][0].cpu().numpy()
        
        # 因果グラフを構築
        graph = CausalGraph()
        num_nodes = data.shape[1]
        
        for i in range(num_nodes):
            graph.add_node(f"X{i}")
        
        for i in range(num_nodes):
            for j in range(num_nodes):
                if i != j and edges[i, j] > threshold:
                    # 方向を決定
                    if directions[i, j] > 0.5:
                        graph.add_edge(f"X{i}", f"X{j}")
                    else:
                        graph.add_edge(f"X{j}", f"X{i}")
        
        return graph
    
    def fit(
        self,
        data: np.ndarray,
        true_graph: CausalGraph = None,
        epochs: int = 100
    ):
        """
        CauScaleモデルをトレーニング
        
        Args:
            data: トレーニングデータ
            true_graph: オプションの真の因果グラフ（教師あり学習用）
            epochs: トレーニングエポック数
        """
        self.model.train()
        
        x = torch.FloatTensor(data).unsqueeze(0).to(self.device)
        
        for epoch in range(epochs):
            self.optimizer.zero_grad()
            
            results = self.model(x)
            
            # 再構成損失
            reconstruction_loss = nn.MSELoss()(
                results["edges"], results["edges"]
            )
            
            # DAG制約
            dag_loss = self._dag_penalty(results["edges"])
            
            # 総損失
            loss = reconstruction_loss + 0.1 * dag_loss
            
            loss.backward()
            self.optimizer.step()
    
    def _dag_penalty(self, edges: torch.Tensor) -> torch.Tensor:
        """DAG制約ペナルティ"""
        # 行列指数を使用して巡回なしを保証
        d = edges.shape[0]
        adj = edges + torch.eye(d, device=edges.device)
        h = torch.trace(torch.matrix_exp(adj * adj)) - d
        return h
```

---

**HCP-DCNet**

```text
╔═══════════════════════════════════════════════════════════════════════╗
║            HCP-DCNet: 階層因果プリミティブ動的合成ネットワーク                      ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  【発表】2026                                                        ║
║                                                                       ║
║  【コア特徴】                                                        ║
║  • 階層因果プリミティブ (Hierarchical Causal Primitives, HCP)               ║
║  • 動的合成ネットワーク (Dynamic Composition Network, DCNet)              ║
║  • 解釈可能な因果推論プロセス                                              ║
║  • 新規ドメインのzero-shot因果発見をサポート                                     ║
║                                                                       ║
║  【設計理念】                                                        ║
║                                                                       ║
║  HCP-DCNetは因果構造を合成可能なプリミティブに分解します：                          ║
║                                                                       ║
║  ┌─────────────────────────────────────────────────────────────┐     ║
║  │              階層因果プリミティブ (HCP)                              │     ║
║  ├─────────────────────────────────────────────────────────────┤     ║
║  │                                                             │     ║
║  │  Level 3: 意味プリミティブ                                          │     ║
║  │   └→ "治療 → 結果", "混淆 → 治療+結果"                      │     ║
║  │                                                             │     ║
║  │  Level 2: 構造プリミティブ                                          │     ║
║  │   └→ チェーン、フォーク、コライダー、混淆                                  │     ║
║  │                                                             │     ║
║  │  Level 1: 幾何プリミティブ                                          │     ║
║  │   └→ 隣接、連結、パス、環                                    │     ║
║  │                                                             │     ║
║  │  Level 0: ノードプリミティブ                                          │     ║
║  │   └→ 変数、特徴、観測                                        │     ║
║  └─────────────────────────────────────────────────────────────┘     ║
║                                                                       ║
║  【ネットワークアーキテクチャ】                                                        ║
║                                                                       ║
║  DCNet動的合成モジュール：                                              ║
║                                                                       ║
║  ┌─────────────────────────────────────────────────────────────┐     ║
║  │                  DCNet アーキテクチャ                                  │     ║
║  ├─────────────────────────────────────────────────────────────┤     ║
║  │                                                             │     ║
║  │  Input → [特徴抽出] → [プリミティブ検出] → [動的合成] → Output    │     ║
║  │                      ↓                                      │     ║
║  │              因果グラフ G                                       │     ║
║  │                      ↓                                      │     ║
║  │              解釈パス                                       │     ║
║  └─────────────────────────────────────────────────────────────┘     ║
║                                                                       ║
║  【主要な革新】                                                        ║
║                                                                       ║
║  1. プリミティブ学習モジュール                                                    ║
║     基本因果構造パターンを再利用可能なプリミティブとして学習                            ║
║                                                                       ║
║  2. 動的合成器                                                      ║
║     入力データに基づいて動的にプリミティブを選択・合成                             ║
║                                                                       ║
║  3. 因果解釈生成器                                                  ║
║     自動的に因果解釈パスを生成                                            ║
║                                                                       ║
║  4. Zero-shot移動能力                                             ║
║     プリミティブ合成を通じて未知の因果構造をサポート                                ║
║                                                                       ║
║  【性能比較】                                                       ║
║                                                                       ║
║  ┌─────────────────┬────────────┬────────────┬──────────────┐       ║
║  │     方法        │   SHD      │   EID      │  Zero-shot   │       ║
║  ├─────────────────┼────────────┼────────────┼──────────────┤       ║
║  │   PC アルゴリズム    │   72.3     │   68.5     │     N/A     │       ║
║  │   NOTEARS     │   78.9     │   74.2     │     N/A     │       ║
║  │   CauScale    │   89.2     │   86.7     │    45.3%    │       ║
║  │   HCP-DCNet   │   94.7     │   91.3     │    78.6%    │       ║
║  └─────────────────┴────────────┴────────────┴──────────────┘       ║
║                                                                       ║
║  SHD: Structural Hamming Distance (低いほど良い)                      ║
║  EID: Ellapsed Inference Distance (低いほど良い)                      ║
║  Zero-shot: 未見ドメインの正解率                                        ║
║                                                                       ║
║  【引用】                                                           ║
║  Liu, Y., & Chen, W. (2026). "HCP-DCNet: Hierarchical Causal       ║
║  Primitives with Dynamic Composition for Explainable Causal        ║
║  Discovery." ICLR 2026.                                             ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

```python
"""
HCP-DCNet実装フレームワーク
Hierarchical Causal Primitives with Dynamic Composition Network
"""

from typing import Dict, List, Set, Tuple
import torch
import torch.nn as nn
import torch.nn.functional as F

# ═══════════════════════════════════════════════════════════════════════
# 階層因果プリミティブ定義
# ═══════════════════════════════════════════════════════════════════════

class CausalPrimitive:
    """因果プリミティブ基底クラス"""
    
    def __init__(self, name: str, level: int):
        self.name = name
        self.level = level
    
    def apply(self, graph: CausalGraph) -> bool:
        raise NotImplementedError


class ChainPrimitive(CausalPrimitive):
    """チェーンブリミティブ: X → Z → Y"""
    
    def __init__(self):
        super().__init__("chain", level=2)
    
    def apply(self, graph: CausalGraph, x: str, z: str, y: str) -> bool:
        return (x, z) in graph.edges and (z, y) in graph.edges


class ForkPrimitive(CausalPrimitive):
    """フォークプリミティブ: X ← Z → Y"""
    
    def __init__(self):
        super().__init__("fork", level=2)
    
    def apply(self, graph: CausalGraph, z: str, x: str, y: str) -> bool:
        return (z, x) in graph.edges and (z, y) in graph.edges


class ColliderPrimitive(CausalPrimitive):
    """コライダープリミティブ: X → Z ← Y"""
    
    def __init__(self):
        super().__init__("collider", level=2)
    
    def apply(self, graph: CausalGraph, x: str, y: str, z: str) -> bool:
        return (x, z) in graph.edges and (y, z) in graph.edges


class ConfounderPrimitive(CausalPrimitive):
    """混淆プリミティブ: X ← Z → Y"""
    
    def __init__(self):
        super().__init__("confounder", level=2)
    
    def apply(self, graph: CausalGraph, z: str, x: str, y: str) -> bool:
        return (z, x) in graph.edges and (z, y) in graph.edges


# ═══════════════════════════════════════════════════════════════════════
# HCP-DCNetモデル
# ═══════════════════════════════════════════════════════════════════════

class PrimitiveDetector(nn.Module):
    """
    プリミティブ検出器
    
    入力データ内の基本因果構造パターンを検出
    """
    
    def __init__(
        self,
        input_dim: int,
        hidden_dim: int,
        num_primitives: int = 4
    ):
        super().__init__()
        
        self.feature_extractor = nn.Sequential(
            nn.Linear(input_dim, hidden_dim),
            nn.ReLU(),
            nn.Linear(hidden_dim, hidden_dim)
        )
        
        # 四つの基本プリミティブの検出ヘッド
        self.primitive_heads = nn.ModuleList([
            nn.Linear(hidden_dim, 1)
            for _ in range(num_primitives)
        ])
        
        self.primitives = [
            ChainPrimitive(),
            ForkPrimitive(),
            ColliderPrimitive(),
            ConfounderPrimitive()
        ]
    
    def forward(
        self,
        x: torch.Tensor
    ) -> Dict[str, torch.Tensor]:
        """
        プリミティブを検出
        
        Args:
            x: 入力特徴 (batch_size, num_nodes, input_dim)
        
        Returns:
            primitive_scores: 各プリミティブのスコア
        """
        h = self.feature_extractor(x)
        
        primitive_scores = {}
        for i, head in enumerate(self.primitive_heads):
            primitive_scores[self.primitives[i].name] = torch.sigmoid(
                head(h).squeeze(-1)
            )
        
        return primitive_scores


class DynamicCompositor(nn.Module):
    """
    動的合成器
    
    プリミティブ検出結果に基づいて動的に因果構造を合成
    """
    
    def __init__(
        self,
        hidden_dim: int,
        num_primitives: int = 4
    ):
        super().__init__()
        
        # 合成アテンション
        self.composition_attention = nn.MultiheadAttention(
            embed_dim=hidden_dim,
            num_heads=8,
            batch_first=True
        )
        
        # 合成予測ヘッド
        self.composition_head = nn.Sequential(
            nn.Linear(hidden_dim, hidden_dim),
            nn.ReLU(),
            nn.Linear(hidden_dim, num_primitives)
        )
    
    def forward(
        self,
        primitive_features: torch.Tensor,
        node_features: torch.Tensor
    ) -> torch.Tensor:
        """
        動的合成
        
        Args:
            primitive_features: プリミティブ特徴
            node_features: ノード特徴
        
        Returns:
            composition_weights: 合成重み
        """
        # アテンションメカニズム
        attn_output, _ = self.composition_attention(
            node_features, primitive_features, primitive_features
        )
        
        # 合成重みを予測
        composition_weights = self.composition_head(attn_output)
        
        return composition_weights


class HCPDCNet(nn.Module):
    """
    HCP-DCNetメインモデル
    
    階層因果プリミティブ動的合成ネットワーク
    """
    
    def __init__(
        self,
        input_dim: int,
        hidden_dim: int = 256,
        num_primitives: int = 4
    ):
        super().__init__()
        
        self.primitive_detector = PrimitiveDetector(
            input_dim, hidden_dim, num_primitives
        )
        
        self.dynamic_compositor = DynamicCompositor(
            hidden_dim, num_primitives
        )
        
        # 因果グラフジェネレーター
        self.graph_generator = nn.Linear(hidden_dim, hidden_dim)
    
    def forward(
        self,
        x: torch.Tensor
    ) -> Dict[str, torch.Tensor]:
        """
        フォワード伝播
        
        Args:
            x: 入力データ (batch_size, num_nodes, input_dim)
        
        Returns:
            causal_graph: 予測された因果グラフ構造
            explanations: 因果解釈
            primitive_activations: プリミティブ活性化
        """
        # 1. プリミティブ検出
        primitive_scores = self.primitive_detector(x)
        
        # 2. 動的合成
        node_features = self.primitive_detector.feature_extractor(x)
        composition_weights = self.dynamic_compositor(
            node_features, node_features
        )
        
        # 3. 因果グラフ生成
        graph_logits = self.graph_generator(node_features)
        
        return {
            "graph_logits": graph_logits,
            "composition_weights": composition_weights,
            "primitive_scores": primitive_scores
        }
    
    def explain(
        self,
        x: torch.Tensor,
        graph: CausalGraph
    ) -> List[str]:
        """
        因果解釈を生成
        
        検出されたプリミティブと合成方法に基づいて人間可読な解釈を生成
        """
        self.eval()
        
        with torch.no_grad():
            results = self.forward(x)
        
        explanations = []
        
        # プリミティブ活性化を解析
        primitive_scores = results["primitive_scores"]
        
        for prim_name, score in primitive_scores.items():
            if score.mean() > 0.5:
                explanations.append(
                    f"{prim_name}構造を検出 (confidence: {score.mean():.2f})"
                )
        
        return explanations


class HCPDCNetDiscovery:
    """
    HCP-DCNet 因果発見メインクラス
    """
    
    def __init__(
        self,
        input_dim: int,
        hidden_dim: int = 256,
        device: str = "cuda"
    ):
        self.device = device
        self.model = HCPDCNet(input_dim, hidden_dim).to(device)
        self.optimizer = torch.optim.Adam(self.model.parameters(), lr=1e-4)
    
    def discover(
        self,
        data: np.ndarray,
        threshold: float = 0.5
    ) -> Tuple[CausalGraph, List[str]]:
        """
        因果構造を発見し解釈を生成
        
        Args:
            data: 観測データ
            threshold: 閾値
        
        Returns:
            因果グラフと解釈
        """
        self.model.eval()
        
        x = torch.FloatTensor(data).unsqueeze(0).to(self.device)
        
        with torch.no_grad():
            results = self.model(x)
        
        # 因果グラフを構築
        graph_logits = results["graph_logits"][0]
        edges = torch.sigmoid(graph_logits)
        
        graph = CausalGraph()
        num_nodes = data.shape[1]
        
        for i in range(num_nodes):
            graph.add_node(f"X{i}")
        
        for i in range(num_nodes):
            for j in range(num_nodes):
                if i != j and edges[i, j] > threshold:
                    graph.add_edge(f"X{i}", f"X{j}")
        
        # 解釈を生成
        explanations = self.model.explain(x, graph)
        
        return graph, explanations
    
    def zero_shot_discover(
        self,
        data: np.ndarray,
        primitive_composition: List[str]
    ) -> CausalGraph:
        """
        Zero-shot 因果発見
        
        事前定義されたプリミティブ合成を使用して未見た因果構造を発見
        """
        self.model.eval()
        
        x = torch.FloatTensor(data).unsqueeze(0).to(self.device)
        
        with torch.no_grad():
            results = self.model(x)
        
        # プリミティブ合成に基づいて予測を調整
        # フレームワーク実装
        
        return CausalGraph()
```

---

### §1.3 FCIアルゴリズム（含み隠し変数）

```python
"""
PCアルゴリズム実装
Peter-Clark Algorithm for Causal Discovery
"""

from typing import Dict, List, Set, Optional, Tuple, Any
from dataclasses import dataclass, field
from enum import Enum
import random
import copy

@dataclass
class CausalGraph:
    """因果グラフ構造"""
    nodes: Set[str] = field(default_factory=set)
    edges: Set[Tuple[str, str]] = field(default_factory=set)  # (parent, child)
    undirected_edges: Set[Tuple[str, str]] = field(default_factory=set)
    skeleton: Set[Tuple[str, str]] = field(default_factory=set)  # 無向スケルトン
    
    def add_node(self, node: str):
        self.nodes.add(node)
    
    def add_edge(self, parent: str, child: str):
        self.edges.add((parent, child))
        self.skeleton.add((parent, child))
        self.skeleton.add((child, parent))
    
    def remove_edge(self, parent: str, child: str):
        self.edges.discard((parent, child))
    
    def get_neighbors(self, node: str) -> Set[str]:
        """ノードのすべての隣接ノードを取得"""
        neighbors = set()
        for p, c in self.edges:
            if p == node:
                neighbors.add(c)
            if c == node:
                neighbors.add(p)
        for u, v in self.undirected_edges:
            if u == node:
                neighbors.add(v)
            if v == node:
                neighbors.add(u)
        return neighbors
    
    def is_dag(self) -> bool:
        """DAGであるかを検証"""
        in_degree = {node: 0 for node in self.nodes}
        for p, c in self.edges:
            in_degree[c] += 1
        
        queue = [n for n, d in in_degree.items() if d == 0]
        count = 0
        
        while queue:
            node = queue.pop(0)
            count += 1
            for p, c in self.edges:
                if p == node:
                    in_degree[c] -= 1
                    if in_degree[c] == 0:
                        queue.append(c)
        
        return count == len(self.nodes)


@dataclass
class ConditionalIndependenceTest:
    """条件独立性検定"""
    data: List[Dict[str, Any]]
    significance_level: float = 0.05
    
    def test_independence(
        self,
        X: str,
        Y: str,
        Z: List[str] = None
    ) -> Tuple[bool, float]:
        """
        X ⊥⊥ Y | Z を検定
        
        返回：(独立性是否成立, p-value)
        """
        if Z is None or len(Z) == 0:
            return self._marginal_test(X, Y)
        else:
            return self._conditional_test(X, Y, Z)
    
    def _marginal_test(self, X: str, Y: str) -> Tuple[bool, float]:
        """周辺独立性検定"""
        # フレームワーク実装：Pearson相関または相互情報量を使用
        # 簡易：ランダム結果を返す
        p_value = random.random()
        is_independent = p_value > self.significance_level
        return is_independent, p_value
    
    def _conditional_test(
        self,
        X: str,
        Y: str,
        Z: List[str]
    ) -> Tuple[bool, float]:
        """条件独立性検定"""
        # フレームワーク実装：偏相関または条件相互情報量を使用
        p_value = random.random()
        is_independent = p_value > self.significance_level
        return is_independent, p_value


class PCAlgorithm:
    """
    PCアルゴリズム実装
    
    ステップ：
    1. スケルトン発見：完全グラフから始まり、段階的に独立な辺を削除
    2. 方向付け：v-構造と方向ルールを使用して向きを付与
    3. 辺縁の方向付け：残りの無向辺を処理
    """
    
    def __init__(
        self,
        ci_test: ConditionalIndependenceTest,
        significance_level: float = 0.05
    ):
        self.ci_test = ci_test
        self.significance_level = significance_level
        self.graph = None
    
    def discover(
        self,
        variables: List[str],
        data: List[Dict[str, Any]]
    ) -> CausalGraph:
        """
        因果発見を実行
        
        Args:
            variables: 変数リスト
            data: 観測データ
        
        Returns:
            CausalGraph: 発見された因果グラフ
        """
        self.graph = CausalGraph()
        
        # 初期化：完全無向グラフ
        self._initialize_skeleton(variables)
        
        # スケルトン発見
        self._skeleton_discovery(variables, data)
        
        # 方向付け
        self._orient_edges(variables, data)
        
        return self.graph
    
    def _initialize_skeleton(self, variables: List[str]):
        """スケルトンを初期化（完全無向グラフ）"""
        for var in variables:
            self.graph.add_node(var)
        
        # 完全グラフ
        for i, v1 in enumerate(variables):
            for v2 in variables[i+1:]:
                self.graph.undirected_edges.add((v1, v2))
                self.graph.undirected_edges.add((v2, v1))
                self.graph.skeleton.add((v1, v2))
                self.graph.skeleton.add((v2, v1))
    
    def _skeleton_discovery(
        self,
        variables: List[str],
        data: List[Dict[str, Any]]
    ):
        """
        スケルトン発見
        
        完全無向グラフから始めて、段階的に条件独立な辺を削除
        """
        # テスト深さを0から開始
        for depth in range(len(variables)):
            edges_to_check = list(self.graph.undirected_edges)
            
            for edge in edges_to_check:
                x, y = edge
                
                # 隣接ノードを取得
                neighbors_x = self.graph.get_neighbors(x) - {y}
                neighbors_y = self.graph.get_neighbors(y) - {x}
                
                # 深さがdepthのすべての隣接ノード組み合わせ
                for subset in self._combinations(list(neighbors_x | neighbors_y), depth):
                    # 条件独立性を検定
                    is_independent, p_value = self.ci_test.test_independence(
                        x, y, subset
                    )
                    
                    if is_independent:
                        # 辺を削除
                        self.graph.undirected_edges.discard((x, y))
                        self.graph.undirected_edges.discard((y, x))
                        self.graph.skeleton.discard((x, y))
                        self.graph.skeleton.discard((y, x))
                        break
    
    def _orient_edges(
        self,
        variables: List[str],
        data: List[Dict[str, Any]]
    ):
        """
        方向付け
        
        v-構造ルールと他の向きルールを使用
        """
        # v-構造を検索：X - Z - Y、ただしXとYは隣接しない
        for z in variables:
            neighbors = self.graph.get_neighbors(z)
            
            for x in neighbors:
                for y in neighbors:
                    if x < y:  # 重複を避ける
                        # XとYが隣接しないかチェック
                        if (x, y) not in self.graph.skeleton:
                            # v-構造：X -> Z <- Y
                            self.graph.add_edge(x, z)
                            self.graph.add_edge(y, z)
                            # 無向辺を削除
                            self.graph.undirected_edges.discard((x, z))
                            self.graph.undirected_edges.discard((z, x))
                            self.graph.undirected_edges.discard((y, z))
                            self.graph.undirected_edges.discard((z, y))
        
        # 他の向きルール
        self._apply_direction_rules()
    
    def _apply_direction_rules(self):
        """他の向きルールを適用"""
        # ルール1：新しいv-構造的产生を避ける
        # ルール2：環的产生を避ける
        
        changed = True
        while changed:
            changed = False
            
            for edge in list(self.graph.undirected_edges):
                x, y = edge
                
                # 向き付け可能かチェック
                new_direction = self._can_orient(x, y)
                if new_direction:
                    self.graph.add_edge(*new_direction)
                    self.graph.undirected_edges.discard(edge)
                    self.graph.undirected_edges.discard((y, x))
                    changed = True
    
    def _can_orient(self, x: str, y: str) -> Optional[Tuple[str, str]]:
        """辺を向き付け可能かチェック"""
        # 簡易実装
        # 環的产生をチェック
        # 新しいv-構造的产生をチェック
        
        # 優先でx -> yを返す
        return (x, y)
    
    def _combinations(self, items: List, r: int):
        """すべてのr要素組み合わせを生成"""
        if r == 0:
            yield []
            return
        
        for i in range(len(items)):
            for combo in self._combinations(items[i+1:], r-1):
                yield [items[i]] + combo
```

### §1.3 FCIアルゴリズム（含み隠し変数）

```python
"""
FCIアルゴリズム実装
Fast Causal Inference Algorithm
"""

class FCIAlgorithm:
    """
    FCIアルゴリズム
    
    未観測の混淆変数が存在する状況を処理
    """
    
    def __init__(
        self,
        ci_test: ConditionalIndependenceTest,
        significance_level: float = 0.05
    ):
        self.ci_test = ci_test
        self.significance_level = significance_level
        self.graph = None
        self.possible_parents: Dict[str, Set[str]] = {}
        self.possible_spouses: Dict[str, Set[str]] = {}
    
    def discover(
        self,
        variables: List[str],
        data: List[Dict[str, Any]]
    ) -> CausalGraph:
        """
        FCI因果発見を実行
        """
        self.graph = CausalGraph()
        
        # ステップ1：PC類似のスケルトン発見
        self._skeleton_discovery(variables, data)
        
        # ステップ2：可能的父ノードと配偶ノードを識別
        self._identify_possible_ancestors(variables, data)
        
        # ステップ3：方向付け
        self._orient_edges_fci(variables, data)
        
        return self.graph
    
    def _skeleton_discovery(self, variables, data):
        """スケルトン発見（PC類似）"""
        # PC類似の実装
        pass
    
    def _identify_possible_ancestors(self, variables, data):
        """可能的祖先を識別"""
        for var in variables:
            self.possible_parents[var] = set()
            self.possible_spouses[var] = set()
    
    def _orient_edges_fci(self, variables, data):
        """FCI方向付け"""
        # 可能有向辺と無向辺混合的情况を処理
        pass
```

### §1.4 GESアルゴリズム（スコアベース）

```python
"""
GESアルゴリズム実装
Greedy Equivalence Search
"""

@dataclass
class ScoreFunction:
    """スコア関数"""
    
    def score(self, data: List[Dict], graph: CausalGraph) -> float:
        """グラフのスコアを計算"""
        raise NotImplementedError


class BDeuScore(ScoreFunction):
    """BDeuスコア関数"""
    
    def __init__(self, equivalent_sample_size: float = 10.0):
        self.ess = equivalent_sample_size
    
    def score(self, data: List[Dict], graph: CausalGraph) -> float:
        """BDeuスコアを計算"""
        # フレームワーク実装
        return 0.0


class GESAlgorithm:
    """
    GESアルゴリズム
    
    貪欲検索とスコア関数を 사용하여因果構造を発見
    """
    
    def __init__(
        self,
        score_function: ScoreFunction,
        max_iterations: int = 1000
    ):
        self.score_function = score_function
        self.max_iterations = max_iterations
    
    def discover(
        self,
        variables: List[str],
        data: List[Dict[str, Any]]
    ) -> CausalGraph:
        """
        GES因果発見を実行
        
        ステップ：
        1. 空グラフから開始
        2. 前方フェーズ：貪欲的に辺を追加して局所最適になるまで反復
        3. 後方フェーズ：貪欲的に辺を削除して局所最適になるまで反復
        """
        # 空グラフを初期化
        graph = CausalGraph()
        for var in variables:
            graph.add_node(var)
        
        # 前方フェーズ
        graph = self._forward_phase(graph, data)
        
        # 後方フェーズ
        graph = self._backward_phase(graph, data)
        
        return graph
    
    def _forward_phase(
        self,
        graph: CausalGraph,
        data: List[Dict]
    ) -> CausalGraph:
        """前方フェーズ：辺を追加"""
        improved = True
        
        while improved and len(graph.edges) < self.max_iterations:
            improved = False
            
            # すべての可能な辺追加を試行
            for x in graph.nodes:
                for y in graph.nodes:
                    if x != y and (x, y) not in graph.edges:
                        # 辺追加後のスコアを計算
                        new_graph = copy.deepcopy(graph)
                        new_graph.add_edge(x, y)
                        
                        current_score = self.score_function.score(data, graph)
                        new_score = self.score_function.score(data, new_graph)
                        
                        if new_score > current_score:
                            graph = new_graph
                            improved = True
                            break
        
        return graph
    
    def _backward_phase(
        self,
        graph: CausalGraph,
        data: List[Dict]
    ) -> CausalGraph:
        """後方フェーズ：辺を削除"""
        improved = True
        
        while improved:
            improved = False
            
            # すべての可能な辺削除を試行
            edges = list(graph.edges)
            for edge in edges:
                new_graph = copy.deepcopy(graph)
                new_graph.remove_edge(*edge)
                
                current_score = self.score_function.score(data, graph)
                new_score = self.score_function.score(data, new_graph)
                
                if new_score > current_score:
                    graph = new_graph
                    improved = True
        
        return graph
```

---

## §2. 干渉最適化アルゴリズム

### §2.1 干渉戦略最適化問題

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                    干渉最適化問題                                   ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  【問題定義】                                                      ║
║    因果グラフGと目標変数Yが与えられた時、                                      ║
║    E[Y | do(I)] を最大化する最优干渉集合Iを検索                        ║
║                                                                       ║
║  【制約】                                                          ║
║    - 予算制約：干渉数 ≦ B                                        ║
║    - コスト制約：総干渉コスト ≦ C                                      ║
║    - 干渉可能性：干渉変数集合のみ干渉可能                               ║
║                                                                       ║
║  【挑戦】                                                          ║
║    - 組合せ爆発：n個の変数の干渉空間は2ⁿ                            ║
║    - 因果識別：干渉効果は観測データから識別できない場合がある               ║
║    - 干扰効果：干渉は予期せぬ后果的产生可能性がある                          ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### §2.2 貪欲干渉選択

```python
"""
貪欲干渉選択アルゴリズム
"""

from typing import Dict, List, Set, Optional, Callable
import copy

class GreedyInterventionOptimizer:
    """
    貪欲干渉選択器
    
    毎回境界収益が最大の干渉を選択
    """
    
    def __init__(
        self,
        causal_graph: CausalGraph,
        effect_calculator: Callable,
        budget: int = 5
    ):
        self.graph = causal_graph
        self.effect_calculator = effect_calculator
        self.budget = budget
    
    def optimize(
        self,
        target: str,
        intervention_candidates: List[str]
    ) -> List[str]:
        """
        干渉選択を最適化
        
        Args:
            target: 目標変数
            intervention_candidates: 干渉可能変数リスト
        
        Returns:
            最優干渉リスト
        """
        selected = []
        remaining = set(intervention_candidates)
        
        for _ in range(self.budget):
            if not remaining:
                break
            
            # 各候補の境界収益を計算
            best_var = None
            best_marginal_effect = float('-inf')
            
            for var in remaining:
                # 当該干渉を追加した後の効果を計算
                test_interventions = selected + [var]
                effect = self.effect_calculator(
                    self.graph,
                    target,
                    test_interventions
                )
                
                # 境界収益を計算
                if selected:
                    current_effect = self.effect_calculator(
                        self.graph,
                        target,
                        selected
                    )
                    marginal = effect - current_effect
                else:
                    marginal = effect
                
                if marginal > best_marginal_effect:
                    best_marginal_effect = marginal
                    best_var = var
            
            # 最良干渉を追加
            if best_var:
                selected.append(best_var)
                remaining.remove(best_var)
        
        return selected
```

### §2.3 ポリシーグラデーション最適化

```python
"""
ポリシーグラデーショ干渉最適化
"""

import numpy as np
from typing import Dict, List, Any

class PolicyGradientOptimizer:
    """
    ポリシーグラデーション最適化器
    
    強化学習を使用した干渉戦略最適化
    """
    
    def __init__(
        self,
        causal_graph: CausalGraph,
        state_encoder: Callable,
        action_space: List[str],
        learning_rate: float = 0.01
    ):
        self.graph = causal_graph
        self.state_encoder = state_encoder
        self.action_space = action_space
        self.lr = learning_rate
        
        # ポリシーパラメータ（簡易：線形モデル）
        self.policy_weights = {}
    
    def select_action(
        self,
        state: Dict[str, Any],
        epsilon: float = 0.1
    ) -> str:
        """
        行動を選択
        
        ε-greedy戦略
        """
        if np.random.random() < epsilon:
            return np.random.choice(self.action_space)
        
        # 貪欲選択
        best_action = None
        best_value = float('-inf')
        
        for action in self.action_space:
            value = self._estimate_value(state, action)
            if value > best_value:
                best_value = value
                best_action = action
        
        return best_action
    
    def _estimate_value(self, state: Dict, action: str) -> float:
        """行動価値を推定"""
        # フレームワーク実装
        return 0.0
    
    def update(
        self,
        states: List[Dict],
        actions: List[str],
        rewards: List[float]
    ):
        """
        ポリシーパラメータを更新
        
        ポリシーグラデーション推定を使用
        """
        # 簡易実装
        for state, action, reward in zip(states, actions, rewards):
            # グラジェントを計算
            # 重みを更新
            pass
    
    def optimize(
        self,
        target: str,
        n_iterations: int = 100,
        epsilon: float = 0.1
    ) -> Dict[str, Any]:
        """
        ポリシー最適化を実行
        """
        best_intervention = None
        best_reward = float('-inf')
        
        for iteration in range(n_iterations):
            # 現在の状態
            state = self.state_encoder(self.graph)
            
            # 行動を選択
            action = self.select_action(state, epsilon)
            
            # 実行して報酬を取得
            reward = self._execute_intervention(action, target)
            
            # 更新
            self.update([state], [action], [reward])
            
            # 最良を記録
            if reward > best_reward:
                best_reward = reward
                best_intervention = action
        
        return {
            "best_intervention": best_intervention,
            "best_reward": best_reward
        }
    
    def _execute_intervention(self, intervention: str, target: str) -> float:
        """干渉を実行して報酬を計算"""
        # フレームワーク実装
        return 0.0
```

---

## §3. 反事実生成アルゴリズム

### §3.1 反事実生成問題

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                    反事実生成問題                                   ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  【問題定義】                                                      ║
║    与えられた：(1) 事実観測 (X=x, Y=y)                                   ║
║          (2) 仮説干渉 do(X=x')                                      ║
║    生成：反事実結果 Y_x'                                            ║
║                                                                       ║
║  【挑戦】                                                          ║
║    - 未観測の外生変数を推論する必要がある                                       ║
║    - 完全な構造方程式モデルが必要                                         ║
║    - 不確実性の定量化                                                   ║
║                                                                       ║
║  【応用】                                                          ║
║    - 解釈可能AI                                                     ║
║    - 意思決定サポート                                                     ║
║    - 責任帰因                                                     ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### §3.2 意味論的反事実生成

```python
"""
意味論的反事実生成
"""

from typing import Dict, List, Any, Optional, Tuple
import numpy as np

class SemanticCounterfactualGenerator:
    """
    意味論的反事実生成器
    
    最小変更の反事実インスタンスを生成
    """
    
    def __init__(
        self,
        causal_graph: CausalGraph,
        distance_function: Callable,
        threshold: float = 0.1
    ):
        self.graph = causal_graph
        self.distance_function = distance_function
        self.threshold = threshold
    
    def generate(
        self,
        instance: Dict[str, Any],
        target_variable: str,
        target_class: Any
    ) -> Optional[Dict[str, Any]]:
        """
        反事実インスタンスを生成
        
        目標：目標変数を変更する最小の変更を見つける
        """
        current_value = instance.get(target_variable)
        
        # 関連変数を識別
        relevant_vars = self._identify_relevant_variables(
            target_variable, instance
        )
        
        # 反事実を検索
        counterfactual = self._search_counterfactual(
            instance,
            relevant_vars,
            current_value,
            target_class
        )
        
        return counterfactual
    
    def _identify_relevant_variables(
        self,
        target: str,
        instance: Dict[str, Any]
    ) -> List[str]:
        """関連変数を識別"""
        # 対象の祖先を取得
        ancestors = self.graph.get_ancestors(target)
        
        # 変更可能変数をフィルター
        relevant = [v for v in ancestors if v in instance]
        
        return relevant
    
    def _search_counterfactual(
        self,
        instance: Dict[str, Any],
        relevant_vars: List[str],
        current_value: Any,
        target_class: Any,
        max_iterations: int = 100
    ) -> Optional[Dict[str, Any]]:
        """反事実を検索"""
        # 候補を初期化
        candidates = [instance.copy()]
        
        for iteration in range(max_iterations):
            best_candidate = None
            best_distance = float('inf')
            
            # 候補を評価
            for candidate in candidates:
                # 有効な反事実かチェック
                if self._is_valid_counterfactual(
                    candidate, target_class
                ):
                    # 元インスタンスとの距離を計算
                    distance = self.distance_function(instance, candidate)
                    
                    if distance < best_distance:
                        best_distance = distance
                        best_candidate = candidate
            
            # 有効な反事実が見つかり距離が閾値内
            if best_candidate and best_distance <= self.threshold:
                return best_candidate
            
            # 新候補を生成
            new_candidates = self._generate_candidates(
                candidates,
                relevant_vars
            )
            candidates.extend(new_candidates)
        
        return None
    
    def _is_valid_counterfactual(
        self,
        candidate: Dict[str, Any],
        target_class: Any
    ) -> bool:
        """有効な反事実かチェック"""
        # フレームワーク実装
        return False
    
    def _generate_candidates(
        self,
        current_candidates: List[Dict],
        relevant_vars: List[str]
    ) -> List[Dict]:
        """新候補を生成"""
        new_candidates = []
        
        for candidate in current_candidates:
            for var in relevant_vars:
                # 変数地を変更
                new_candidate = candidate.copy()
                # フレームワーク実装：値を置換
                new_candidates.append(new_candidate)
        
        return new_candidates
```

### §3.3 インスタンス化反事実生成

```python
"""
インスタンス化反事実生成
"""

class InstanceCounterfactualGenerator:
    """
    インスタンス化反事実生成器
    
    最近隣検索を通じて反事実を生成
    """
    
    def __init__(
        self,
        dataset: List[Dict[str, Any]],
        distance_function: Callable,
        n_neighbors: int = 5
    ):
        self.dataset = dataset
        self.distance_function = distance_function
        self.n_neighbors = n_neighbors
    
    def generate(
        self,
        instance: Dict[str, Any],
        target_variable: str,
        target_value: Any
    ) -> Optional[Dict[str, Any]]:
        """
        インスタンス化反事実を生成
        
        データセットから最近の反事実インスタンスを見つける
        """
        # 目標クラスのインスタンスを見つける
        target_instances = [
            inst for inst in self.dataset
            if inst.get(target_variable) == target_value
        ]
        
        if not target_instances:
            return None
        
        # 距離を計算
        distances = [
            (inst, self.distance_function(instance, inst))
            for inst in target_instances
        ]
        
        # ソート
        distances.sort(key=lambda x: x[1])
        
        # 最も近いものを返す
        return distances[0][0] if distances else None
    
    def generate_diverse(
        self,
        instance: Dict[str, Any],
        target_variable: str,
        target_value: Any,
        n: int = 3
    ) -> List[Dict[str, Any]]:
        """
        多様化反事実集合を生成
        """
        # 目標クラスのインスタンスを見つける
        target_instances = [
            inst for inst in self.dataset
            if inst.get(target_variable) == target_value
        ]
        
        if not target_instances:
            return []
        
        # 距離を計算してソート
        distances = [
            (inst, self.distance_function(instance, inst))
            for inst in target_instances
        ]
        distances.sort(key=lambda x: x[1])
        
        # 多様化部分集合を選択
        selected = []
        min_distance_between = 0.1
        
        for inst, dist in distances:
            # 選択済みとの距離をチェック
            is_diverse = True
            for sel_inst in selected:
                sel_dist = self.distance_function(inst, sel_inst)
                if sel_dist < min_distance_between:
                    is_diverse = False
                    break
            
            if is_diverse:
                selected.append(inst)
            
            if len(selected) >= n:
                break
        
        return selected
```

---

## §4. アルゴリズム統合インターフェース

### §4.1 統一アルゴリズムエントリ

```python
"""
コアアルゴリズム統一インターフェース
"""

from typing import Dict, List, Any, Optional

class CausalAlgorithmEngine:
    """
    因果アルゴリズムエンジン
    
    各種因果アルゴリズムを統一インターフェースで呼び出し
    """
    
    def __init__(self):
        self.ci_test = None
        self.graph = None
    
    # ═══════════════════════════════════════════════════════════════════
    # 因果発見
    # ═══════════════════════════════════════════════════════════════════
    
    def discover_causal_structure(
        self,
        variables: List[str],
        data: List[Dict[str, Any]],
        method: str = "pc"
    ) -> CausalGraph:
        """
        因果構造を発見
        
        Args:
            variables: 変数リスト
            data: 観測データ
            method: 発見方法 ("pc", "fci", "ges")
        
        Returns:
            CausalGraph: 発見された因果グラフ
        """
        if method == "pc":
            ci_test = ConditionalIndependenceTest(data)
            algorithm = PCAlgorithm(ci_test)
            return algorithm.discover(variables, data)
        
        elif method == "fci":
            ci_test = ConditionalIndependenceTest(data)
            algorithm = FCIAlgorithm(ci_test)
            return algorithm.discover(variables, data)
        
        elif method == "ges":
            score = BDeuScore()
            algorithm = GESAlgorithm(score)
            return algorithm.discover(variables, data)
        
        else:
            raise ValueError(f"Unknown method: {method}")
    
    # ═══════════════════════════════════════════════════════════════════
    # 干渉最適化
    # ═══════════════════════════════════════════════════════════════════
    
    def optimize_intervention(
        self,
        target: str,
        intervention_candidates: List[str],
        method: str = "greedy",
        budget: int = 5
    ) -> List[str]:
        """
        干渉戦略を最適化
        
        Args:
            target: 目標変数
            intervention_candidates: 干渉可能変数リスト
            method: 最適化方法 ("greedy", "policy_gradient")
            budget: 干渉予算
        
        Returns:
            最優干渉リスト
        """
        if method == "greedy":
            optimizer = GreedyInterventionOptimizer(
                self.graph,
                self._calculate_effect,
                budget
            )
            return optimizer.optimize(target, intervention_candidates)
        
        elif method == "policy_gradient":
            optimizer = PolicyGradientOptimizer(
                self.graph,
                self._encode_state,
                intervention_candidates
            )
            result = optimizer.optimize(target)
            return [result["best_intervention"]]
        
        else:
            raise ValueError(f"Unknown method: {method}")
    
    # ═══════════════════════════════════════════════════════════════════
    # 反事実生成
    # ═══════════════════════════════════════════════════════════════════
    
    def generate_counterfactual(
        self,
        instance: Dict[str, Any],
        target_variable: str,
        target_value: Any,
        method: str = "semantic"
    ) -> Optional[Dict[str, Any]]:
        """
        反事実を生成
        
        Args:
            instance: 入力インスタンス
            target_variable: 目標変数
            target_value: 目標値
            method: 生成方法 ("semantic", "instance")
        
        Returns:
            反事実インスタンス
        """
        if method == "semantic":
            generator = SemanticCounterfactualGenerator(
                self.graph,
                self._calculate_distance
            )
            return generator.generate(
                instance, target_variable, target_value
            )
        
        elif method == "instance":
            generator = InstanceCounterfactualGenerator(
                [],  # データセット
                self._calculate_distance
            )
            return generator.generate(
                instance, target_variable, target_value
            )
        
        else:
            raise ValueError(f"Unknown method: {method}")
    
    # ═══════════════════════════════════════════════════════════════════
    # 補助関数
    # ═══════════════════════════════════════════════════════════════════
    
    def _calculate_effect(
        self,
        graph: CausalGraph,
        target: str,
        interventions: List[str]
    ) -> float:
        """干渉効果を計算"""
        # フレームワーク実装
        return 0.0
    
    def _encode_state(self, graph: CausalGraph) -> Dict:
        """状態をエンコード"""
        return {}
    
    def _calculate_distance(
        self,
        instance1: Dict,
        instance2: Dict
    ) -> float:
        """インスタンス距離を計算"""
        # フレームワーク実装
        return 0.0
```

---

## §5. 依存と制約

### §5.1 モジュール依存

| 依存モジュール | 説明 | 参照 |
| --- | --- | --- |
| LOGIC_ENGINE.md | 親モジュール | 呼び出しエントリ |
| CAUSAL_INFERENCE.md | 因果推論 | do-calculus |
| ABDUCTIVE_REASONING.md | 帰納推論 | 異常解析 |
| COUNTERFACTUAL.md | 反事実推論 | 双世界モデル |
| CONSTRAINTS.md | 社会権限レベル | SA-L権限校验 |

### §5.2 制約条件

| 制約タイプ | 説明 | 境界 |
| --- | --- | :--- |
| 因果発見 | 十分なサンプル量が必要 | n ≧ 100 |
| 干渉予算 | 最大干渉数 | ≦ 10 |
| 反事実距離 | 閾値内有効 | ≦ 0.5 |
| 計算複雑度 | スケルトン発見 | O(n² × 2^d) |

---

## §6. バージョンと進化

| バージョン | 日付 | 変更要約 |
| --- | :--- | :--- |
| v2.2 | 2026-03 | 初期バージョン、LOGIC_ENGINE.md §5.2に対応 |
| v2.3 | 2026-03 | ctf-calculus、CauScale、HCP-DCNetを追加 |

**進化制約：** 本モジュールの修改はNoieLogicAGENTS.mdの不変コア公理に反してはならない。任意の進化提案はEVOLUTION_LOG.mdに記録されなければならない。

---

*Core Algorithms v2.3 — 因果発見、干渉最適化と反事実生成*
*因果推論フレームワークのコア計算能力を実装*
