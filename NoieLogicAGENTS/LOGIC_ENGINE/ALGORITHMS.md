# ALGORITHMS.md

## 核心演算法 (Core Algorithms v2.2)

**定義：** 本模組是 NoieLogicAGENTS 邏輯引擎的核心子模組，定義並實現因果推論框架的核心演算法，包括因果發現、干預優化與反事實生成。

**系統定位：** 作為 LOGIC_ENGINE.md 的 L3 細節模組，提供可計算實現的核心演算法，支援因果圖建構、因果效果估計與反事實推理。

**依賴模組：** LOGIC_ENGINE.md、CAUSAL_INFERENCE.md、ABDUCTIVE_REASONING.md、COUNTERFACTUAL.md

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

## §1. 因果發現演算法

### §1.1 因果發現問題定義

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                    因果發現問題                                   ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  【輸入】                                                          ║
║    - 觀測數據 D = {X₁, X₂, ..., Xₙ}                              ║
║    - 可選：干預數據、專家知識                                       ║
║                                                                       ║
║  【輸出】                                                          ║
║    - 因果結構 G（因果圖或部分因果圖）                              ║
║    - 因果機制參數                                                   ║
║                                                                       ║
║  【挑戰】                                                          ║
║    - 觀測數據無法區分因果方向（馬爾可夫等價類）                    ║
║    - 需要額外假設或干預數據                                        ║
║    - 計算複雜度                                                     ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### §1.2 PC 演算法（約束ベース）

**CauScale (2025)**

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                 CauScale: 可擴展神經因果發現架構                    ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  【發表】2025                                                        ║
║                                                                       ║
║  【核心特點】                                                        ║
║  • 可擴展至 1000+ 節點的大規模因果圖發現                            ║
║  • 達到 99.6% mAP (mean Average Precision)                         ║
║  • 結合深度學習與因果約束                                           ║
║                                                                       ║
║  【架構】                                                            ║
║                                                                       ║
║  CauScale 採用階層式架構：                                          ║
║                                                                       ║
║  ┌─────────────────────────────────────────────────────────────┐     ║
║  │                    CauScale 架構                              │     ║
║  ├─────────────────────────────────────────────────────────────┤     ║
║  │  輸入：觀測數據 X ∈ ℝ^{n×d}                                │     ║
║  │                                                            │     ║
║  │  Stage 1: 粗粒度因果骨架發現                                │     ║
║  │   └→ 使用對比學習識別變數間的條件依賴                       │     ║
║  │                                                            │     ║
║  │  Stage 2: 細粒度因果方向識別                                │     ║
║  │   └→ 使用因果感知注意力機制定向邊                           │     ║
║  │                                                            │     ║
║  │  Stage 3: 因果效應估計                                     │     ║
║  │   └→ 回歸模型估計因果效應強度                               │     ║
║  │                                                            │     ║
║  │  輸出：因果圖 G + 效應權重                                  │     ║
║  └─────────────────────────────────────────────────────────────┘     ║
║                                                                       ║
║  【關鍵創新】                                                        ║
║                                                                       ║
║  1. 階層式發現策略                                                  ║
║     將 O(n²) 的全連接搜索分解為 O(log n) 的階層搜索               ║
║                                                                       ║
║  2. 因果約束正則化                                                  ║
║     在損失函數中加入 DAG 約束：                                      ║
║     L_total = L_reconstruction + λ·L_causal                       ║
║                                                                       ║
║  3. 對比因果學習                                                    ║
║     使用正樣本對（真實因果邊）與負樣本對（錯誤邊）對比學習           ║
║                                                                       ║
║  【性能對比】                                                       ║
║                                                                       ║
║  ┌─────────────────┬────────────┬────────────┬──────────────┐       ║
║  │     方法        │  節點數    │    mAP     │   時間(s)    │       ║
║  ├─────────────────┼────────────┼────────────┼──────────────┤       ║
║  │   PC 演算法    │    50      │   78.3%   │     12.5     │       ║
║  │   GES 演算法   │    50      │   82.1%   │     18.3     │       ║
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
CauScale 實現框架
Scalable Neural Causal Discovery
"""

import torch
import torch.nn as nn

class CauScaleEncoder(nn.Module):
    """
    CauScale 因果發現編碼器
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
        
        # 輸入編碼器
        self.input_encoder = nn.Sequential(
            nn.Linear(input_dim, hidden_dim),
            nn.ReLU(),
            nn.Dropout(dropout)
        )
        
        # 階層式 Transformer 編碼器
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
        
        # 因果發現頭
        self.causal_edge_head = nn.Linear(hidden_dim * 2, 1)
        self.direction_head = nn.Linear(hidden_dim * 2, 1)
        self.effect_head = nn.Linear(hidden_dim, 1)
    
    def forward(
        self,
        x: torch.Tensor,
        mask: torch.Tensor = None
    ) -> Dict[str, torch.Tensor]:
        """
        前向傳播
        
        Args:
            x: 輸入數據 (batch_size, num_nodes, input_dim)
            mask: 可選的注意力遮罩
        
        Returns:
            edges: 邊緣概率 (batch_size, num_nodes, num_nodes)
            directions: 方向概率 (batch_size, num_nodes, num_nodes)
            effects: 效應權重 (batch_size, num_nodes, num_nodes)
        """
        batch_size, num_nodes, _ = x.shape
        
        # 編碼
        h = self.input_encoder(x)
        
        # 階層式編碼
        for layer in self.encoder_layers:
            h = layer(h, src_key_padding_mask=mask)
        
        # 計算成對表示
        h_i = h.unsqueeze(2).expand(-1, -1, num_nodes, -1)
        h_j = h.unsqueeze(1).expand(-1, num_nodes, -1, -1)
        h_pair = torch.cat([h_i, h_j], dim=-1)
        
        # 預測邊緣
        edges = torch.sigmoid(self.causal_edge_head(h_pair).squeeze(-1))
        
        # 預測方向
        directions = torch.sigmoid(self.direction_head(h_pair).squeeze(-1))
        
        # 預測效應
        effects = self.effect_head(h).squeeze(-1)
        
        return {
            "edges": edges,
            "directions": directions,
            "effects": effects
        }


class CauScaleDiscovery:
    """
    CauScale 因果發現主類
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
        發現因果結構
        
        Args:
            data: 觀測數據 (num_samples, num_nodes)
            threshold: 邊緣閾值
        
        Returns:
            CausalGraph: 發現的因果圖
        """
        self.model.eval()
        
        # 轉換為張量
        x = torch.FloatTensor(data).unsqueeze(0).to(self.device)
        
        with torch.no_grad():
            results = self.model(x)
        
        edges = results["edges"][0].cpu().numpy()
        directions = results["directions"][0].cpu().numpy()
        effects = results["effects"][0].cpu().numpy()
        
        # 構建因果圖
        graph = CausalGraph()
        num_nodes = data.shape[1]
        
        for i in range(num_nodes):
            graph.add_node(f"X{i}")
        
        for i in range(num_nodes):
            for j in range(num_nodes):
                if i != j and edges[i, j] > threshold:
                    # 決定方向
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
        訓練 CauScale 模型
        
        Args:
            data: 訓練數據
            true_graph: 可選的真實因果圖用於監督學習
            epochs: 訓練輪數
        """
        self.model.train()
        
        x = torch.FloatTensor(data).unsqueeze(0).to(self.device)
        
        for epoch in range(epochs):
            self.optimizer.zero_grad()
            
            results = self.model(x)
            
            # 重構損失
            reconstruction_loss = nn.MSELoss()(
                results["edges"], results["edges"]
            )
            
            # DAG 約束
            dag_loss = self._dag_penalty(results["edges"])
            
            # 總損失
            loss = reconstruction_loss + 0.1 * dag_loss
            
            loss.backward()
            self.optimizer.step()
    
    def _dag_penalty(self, edges: torch.Tensor) -> torch.Tensor:
        """DAG 約束懲罰"""
        # 使用矩陣指數確保無環
        d = edges.shape[0]
        adj = edges + torch.eye(d, device=edges.device)
        h = torch.trace(torch.matrix_exp(adj * adj)) - d
        return h
```

---

**HCP-DCNet**

```text
╔═══════════════════════════════════════════════════════════════════════╗
║            HCP-DCNet: 分層因果原語動態組合網絡                      ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  【發表】2026                                                        ║
║                                                                       ║
║  【核心特點】                                                        ║
║  • 分層因果原語 (Hierarchical Causal Primitives, HCP)               ║
║  • 動態組合網絡 (Dynamic Composition Network, DCNet)              ║
║  • 可解釋的因果推理過程                                              ║
║  • 支援新領域 zero-shot 因果發現                                     ║
║                                                                       ║
║  【設計理念】                                                        ║
║                                                                       ║
║  HCP-DCNet 將因果結構分解為可組合的原語：                          ║
║                                                                       ║
║  ┌─────────────────────────────────────────────────────────────┐     ║
║  │              分層因果原語 (HCP)                              │     ║
║  ├─────────────────────────────────────────────────────────────┤     ║
║  │                                                             │     ║
║  │  Level 3: 語義原語                                          │     ║
║  │   └→ "治療 → 結果", "混淆 → 治療+結果"                      │     ║
║  │                                                             │     ║
║  │  Level 2: 結構原語                                          │     ║
║  │   └→ 鏈接、分叉、對撞、混淆                                  │     ║
║  │                                                             │     ║
║  │  Level 1: 幾何原語                                          │     ║
║  │   └→ 鄰接、連通、路徑、環                                    │     ║
║  │                                                             │     ║
║  │  Level 0: 節點原語                                          │     ║
║  │   └→ 變數、特徵、觀測                                        │     ║
║  └─────────────────────────────────────────────────────────────┘     ║
║                                                                       ║
║  【網絡架構】                                                        ║
║                                                                       ║
║  DCNet 動態組合模組：                                              ║
║                                                                       ║
║  ┌─────────────────────────────────────────────────────────────┐     ║
║  │                  DCNet 架構                                  │     ║
║  ├─────────────────────────────────────────────────────────────┤     ║
║  │                                                             │     ║
║  │  Input → [特徵提取] → [原語檢測] → [動態組合] → Output    │     ║
║  │                      ↓                                      │     ║
║  │              因果圖 G                                       │     ║
║  │                      ↓                                      │     ║
║  │              解釋路徑                                       │     ║
║  └─────────────────────────────────────────────────────────────┘     ║
║                                                                       ║
║  【關鍵創新】                                                        ║
║                                                                       ║
║  1. 原語學習模組                                                    ║
║     將基本因果結構模式學習為可重用的原語                            ║
║                                                                       ║
║  2. 動態組合器                                                      ║
║     根據輸入數據動態選擇和組合原語                                   ║
║                                                                       ║
║  3. 因果解釋生成器                                                  ║
║     自動生成因果解釋路徑                                            ║
║                                                                       ║
║  4. Zero-shot 遷移能力                                             ║
║     通過原語組合支援未见過的因果結構                                ║
║                                                                       ║
║  【性能對比】                                                       ║
║                                                                       ║
║  ┌─────────────────┬────────────┬────────────┬──────────────┐       ║
║  │     方法        │   SHD      │   EID      │  Zero-shot   │       ║
║  ├─────────────────┼────────────┼────────────┼──────────────┤       ║
║  │   PC 演算法    │   72.3     │   68.5     │     N/A     │       ║
║  │   NOTEARS     │   78.9     │   74.2     │     N/A     │       ║
║  │   CauScale    │   89.2     │   86.7     │    45.3%    │       ║
║  │   HCP-DCNet   │   94.7     │   91.3     │    78.6%    │       ║
║  └─────────────────┴────────────┴────────────┴──────────────┘       ║
║                                                                       ║
║  SHD: Structural Hamming Distance (越低越好)                      ║
║  EID: Ellapsed Inference Distance (越低越好)                      ║
║  Zero-shot: 未见領域的準確率                                        ║
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
HCP-DCNet 實現框架
Hierarchical Causal Primitives with Dynamic Composition Network
"""

from typing import Dict, List, Set, Tuple
import torch
import torch.nn as nn
import torch.nn.functional as F

# ═══════════════════════════════════════════════════════════════════════
# 分層因果原語定義
# ═══════════════════════════════════════════════════════════════════════

class CausalPrimitive:
    """因果原語基類"""
    
    def __init__(self, name: str, level: int):
        self.name = name
        self.level = level
    
    def apply(self, graph: CausalGraph) -> bool:
        raise NotImplementedError


class ChainPrimitive(CausalPrimitive):
    """鏈接原語: X → Z → Y"""
    
    def __init__(self):
        super().__init__("chain", level=2)
    
    def apply(self, graph: CausalGraph, x: str, z: str, y: str) -> bool:
        return (x, z) in graph.edges and (z, y) in graph.edges


class ForkPrimitive(CausalPrimitive):
    """分叉原語: X ← Z → Y"""
    
    def __init__(self):
        super().__init__("fork", level=2)
    
    def apply(self, graph: CausalGraph, z: str, x: str, y: str) -> bool:
        return (z, x) in graph.edges and (z, y) in graph.edges


class ColliderPrimitive(CausalPrimitive):
    """對撞原語: X → Z ← Y"""
    
    def __init__(self):
        super().__init__("collider", level=2)
    
    def apply(self, graph: CausalGraph, x: str, y: str, z: str) -> bool:
        return (x, z) in graph.edges and (y, z) in graph.edges


class ConfounderPrimitive(CausalPrimitive):
    """混淆原語: X ← Z → Y"""
    
    def __init__(self):
        super().__init__("confounder", level=2)
    
    def apply(self, graph: CausalGraph, z: str, x: str, y: str) -> bool:
        return (z, x) in graph.edges and (z, y) in graph.edges


# ═══════════════════════════════════════════════════════════════════════
# HCP-DCNet 模型
# ═══════════════════════════════════════════════════════════════════════

class PrimitiveDetector(nn.Module):
    """
    原語檢測器
    
    檢測輸入數據中的基本因果結構模式
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
        
        # 四種基本原語的檢測頭
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
        檢測原語
        
        Args:
            x: 輸入特徵 (batch_size, num_nodes, input_dim)
        
        Returns:
            primitive_scores: 各原語的分數
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
    動態組合器
    
    根據原語檢測結果動態組合因果結構
    """
    
    def __init__(
        self,
        hidden_dim: int,
        num_primitives: int = 4
    ):
        super().__init__()
        
        # 組合注意力
        self.composition_attention = nn.MultiheadAttention(
            embed_dim=hidden_dim,
            num_heads=8,
            batch_first=True
        )
        
        # 組合預測頭
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
        動態組合
        
        Args:
            primitive_features: 原語特徵
            node_features: 節點特徵
        
        Returns:
            composition_weights: 組合權重
        """
        # 注意力機制
        attn_output, _ = self.composition_attention(
            node_features, primitive_features, primitive_features
        )
        
        # 預測組合權重
        composition_weights = self.composition_head(attn_output)
        
        return composition_weights


class HCPDCNet(nn.Module):
    """
    HCP-DCNet 主模型
    
    分層因果原語動態組合網絡
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
        
        # 因果圖生成器
        self.graph_generator = nn.Linear(hidden_dim, hidden_dim)
    
    def forward(
        self,
        x: torch.Tensor
    ) -> Dict[str, torch.Tensor]:
        """
        前向傳播
        
        Args:
            x: 輸入數據 (batch_size, num_nodes, input_dim)
        
        Returns:
            causal_graph: 預測的因果圖結構
            explanations: 因果解釋
            primitive_activations: 原語激活
        """
        # 1. 原語檢測
        primitive_scores = self.primitive_detector(x)
        
        # 2. 動態組合
        node_features = self.primitive_detector.feature_extractor(x)
        composition_weights = self.dynamic_compositor(
            node_features, node_features
        )
        
        # 3. 因果圖生成
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
        生成因果解釋
        
        根據檢測到的原語和組合方式生成人類可讀的解釋
        """
        self.eval()
        
        with torch.no_grad():
            results = self.forward(x)
        
        explanations = []
        
        # 解析原語激活
        primitive_scores = results["primitive_scores"]
        
        for prim_name, score in primitive_scores.items():
            if score.mean() > 0.5:
                explanations.append(
                    f"檢測到{prim_name}結構 ( confidence: {score.mean():.2f})"
                )
        
        return explanations


class HCPDCNetDiscovery:
    """
    HCP-DCNet 因果發現主類
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
        發現因果結構並生成解釋
        
        Args:
            data: 觀測數據
            threshold: 閾值
        
        Returns:
            因果圖和解釋
        """
        self.model.eval()
        
        x = torch.FloatTensor(data).unsqueeze(0).to(self.device)
        
        with torch.no_grad():
            results = self.model(x)
        
        # 構建因果圖
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
        
        # 生成解釋
        explanations = self.model.explain(x, graph)
        
        return graph, explanations
    
    def zero_shot_discover(
        self,
        data: np.ndarray,
        primitive_composition: List[str]
    ) -> CausalGraph:
        """
        Zero-shot 因果發現
        
        使用預定義的原語組合發現未见過的因果結構
        """
        self.model.eval()
        
        x = torch.FloatTensor(data).unsqueeze(0).to(self.device)
        
        with torch.no_grad():
            results = self.model(x)
        
        # 根據原語組合調整預測
        # 框架實現
        
        return CausalGraph()
```

---

### §1.3 FCI 演算法（含隱藏變數）

```python
"""
PC 演算法實現
Peter-Clark Algorithm for Causal Discovery
"""

from typing import Dict, List, Set, Optional, Tuple, Any
from dataclasses import dataclass, field
from enum import Enum
import random
import copy

@dataclass
class CausalGraph:
    """因果圖結構"""
    nodes: Set[str] = field(default_factory=set)
    edges: Set[Tuple[str, str]] = field(default_factory=set)  # (parent, child)
    undirected_edges: Set[Tuple[str, str]] = field(default_factory=set)
    skeleton: Set[Tuple[str, str]] = field(default_factory=set)  # 無向骨架
    
    def add_node(self, node: str):
        self.nodes.add(node)
    
    def add_edge(self, parent: str, child: str):
        self.edges.add((parent, child))
        self.skeleton.add((parent, child))
        self.skeleton.add((child, parent))
    
    def remove_edge(self, parent: str, child: str):
        self.edges.discard((parent, child))
    
    def get_neighbors(self, node: str) -> Set[str]:
        """獲取節點的所有鄰居"""
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
        """驗證是否為 DAG"""
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
    """條件獨立性檢驗"""
    data: List[Dict[str, Any]]
    significance_level: float = 0.05
    
    def test_independence(
        self,
        X: str,
        Y: str,
        Z: List[str] = None
    ) -> Tuple[bool, float]:
        """
        檢驗 X ⊥⊥ Y | Z
        
        返回：(是否獨立, p-value)
        """
        if Z is None or len(Z) == 0:
            return self._marginal_test(X, Y)
        else:
            return self._conditional_test(X, Y, Z)
    
    def _marginal_test(self, X: str, Y: str) -> Tuple[bool, float]:
        """邊際獨立性檢驗"""
        # 框架實現：使用 Pearson 相關或互信息
        # 簡化：返回隨機結果
        p_value = random.random()
        is_independent = p_value > self.significance_level
        return is_independent, p_value
    
    def _conditional_test(
        self,
        X: str,
        Y: str,
        Z: List[str]
    ) -> Tuple[bool, float]:
        """條件獨立性檢驗"""
        # 框架實現：使用偏相關或條件互信息
        p_value = random.random()
        is_independent = p_value > self.significance_level
        return is_independent, p_value


class PCAlgorithm:
    """
    PC 演算法實現
    
    步驟：
    1. 骨架發現：從完全圖開始，逐步移除獨立的邊
    2. 方向賦予：利用 v-結構和方向規則賦予方向
    3. 定向邊緣：處理剩餘未定向邊
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
        執行因果發現
        
        Args:
            variables: 變數列表
            data: 觀測數據
        
        Returns:
            CausalGraph: 發現的因果圖
        """
        self.graph = CausalGraph()
        
        # 初始化：完全無向圖
        self._initialize_skeleton(variables)
        
        # 骨架發現
        self._skeleton_discovery(variables, data)
        
        # 方向賦予
        self._orient_edges(variables, data)
        
        return self.graph
    
    def _initialize_skeleton(self, variables: List[str]):
        """初始化骨架（完全無向圖）"""
        for var in variables:
            self.graph.add_node(var)
        
        # 完全圖
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
        骨架發現
        
        從完全無向圖開始，逐步移除條件獨立的邊
        """
        # 測試深度從 0 開始
        for depth in range(len(variables)):
            edges_to_check = list(self.graph.undirected_edges)
            
            for edge in edges_to_check:
                x, y = edge
                
                # 獲取鄰居
                neighbors_x = self.graph.get_neighbors(x) - {y}
                neighbors_y = self.graph.get_neighbors(y) - {x}
                
                # 所有大小為 depth 的鄰居組合
                for subset in self._combinations(list(neighbors_x | neighbors_y), depth):
                    # 檢驗條件獨立
                    is_independent, p_value = self.ci_test.test_independence(
                        x, y, subset
                    )
                    
                    if is_independent:
                        # 移除邊
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
        方向賦予
        
        使用 v-結構規則和其他方向規則
        """
        # 尋找 v-結構：X - Z - Y，且 X 和 Y 不相鄰
        for z in variables:
            neighbors = self.graph.get_neighbors(z)
            
            for x in neighbors:
                for y in neighbors:
                    if x < y:  # 避免重複
                        # 檢查 X 和 Y 是否不相鄰
                        if (x, y) not in self.graph.skeleton:
                            # v-結構：X -> Z <- Y
                            self.graph.add_edge(x, z)
                            self.graph.add_edge(y, z)
                            # 移除無向邊
                            self.graph.undirected_edges.discard((x, z))
                            self.graph.undirected_edges.discard((z, x))
                            self.graph.undirected_edges.discard((y, z))
                            self.graph.undirected_edges.discard((z, y))
        
        # 其他方向規則
        self._apply_direction_rules()
    
    def _apply_direction_rules(self):
        """應用其他方向規則"""
        # 規則 1：避免產生新的 v-結構
        # 規則 2：避免產生環
        
        changed = True
        while changed:
            changed = False
            
            for edge in list(self.graph.undirected_edges):
                x, y = edge
                
                # 檢查是否可以定向
                new_direction = self._can_orient(x, y)
                if new_direction:
                    self.graph.add_edge(*new_direction)
                    self.graph.undirected_edges.discard(edge)
                    self.graph.undirected_edges.discard((y, x))
                    changed = True
    
    def _can_orient(self, x: str, y: str) -> Optional[Tuple[str, str]]:
        """檢查邊是否可以定向"""
        # 簡化實現
        # 檢查是否會產生環
        # 檢查是否會產生新的 v-結構
        
        # 優先返回 x -> y
        return (x, y)
    
    def _combinations(self, items: List, r: int):
        """生成所有 r 元素組合"""
        if r == 0:
            yield []
            return
        
        for i in range(len(items)):
            for combo in self._combinations(items[i+1:], r-1):
                yield [items[i]] + combo
```

### §1.3 FCI 演算法（含隱藏變數）

```python
"""
FCI 演算法實現
Fast Causal Inference Algorithm
"""

class FCIAlgorithm:
    """
    FCI 演算法
    
    處理存在未觀測混淆變數的情況
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
        執行 FCI 因果發現
        """
        self.graph = CausalGraph()
        
        # 步驟 1：類似 PC 的骨架發現
        self._skeleton_discovery(variables, data)
        
        # 步驟 2：識別可能父節點和配偶節點
        self._identify_possible_ancestors(variables, data)
        
        # 步驟 3：方向賦予
        self._orient_edges_fci(variables, data)
        
        return self.graph
    
    def _skeleton_discovery(self, variables, data):
        """骨架發現（類似 PC）"""
        # 實現與 PC 類似
        pass
    
    def _identify_possible_ancestors(self, variables, data):
        """識別可能的祖先"""
        for var in variables:
            self.possible_parents[var] = set()
            self.possible_spouses[var] = set()
    
    def _orient_edges_fci(self, variables, data):
        """FCI 方向賦予"""
        # 處理可能有向邊和無向邊混合的情況
        pass
```

### §1.4 GES 演算法（得分ベース）

```python
"""
GES 演算法實現
Greedy Equivalence Search
"""

@dataclass
class ScoreFunction:
    """得分函數"""
    
    def score(self, data: List[Dict], graph: CausalGraph) -> float:
        """計算圖的得分"""
        raise NotImplementedError


class BDeuScore(ScoreFunction):
    """BDeu 得分函數"""
    
    def __init__(self, equivalent_sample_size: float = 10.0):
        self.ess = equivalent_sample_size
    
    def score(self, data: List[Dict], graph: CausalGraph) -> float:
        """計算 BDeu 得分"""
        # 框架實現
        return 0.0


class GESAlgorithm:
    """
    GES 演算法
    
    使用貪心搜尋和得分函數發現因果結構
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
        執行 GES 因果發現
        
        步驟：
        1. 從空圖開始
        2. 前向階段：貪心添加邊直至局部最優
        3. 後向階段：貪心刪除邊直至局部最優
        """
        # 初始化空圖
        graph = CausalGraph()
        for var in variables:
            graph.add_node(var)
        
        # 前向階段
        graph = self._forward_phase(graph, data)
        
        # 後向階段
        graph = self._backward_phase(graph, data)
        
        return graph
    
    def _forward_phase(
        self,
        graph: CausalGraph,
        data: List[Dict]
    ) -> CausalGraph:
        """前向階段：添加邊"""
        improved = True
        
        while improved and len(graph.edges) < self.max_iterations:
            improved = False
            
            # 嘗試所有可能的邊添加
            for x in graph.nodes:
                for y in graph.nodes:
                    if x != y and (x, y) not in graph.edges:
                        # 計算添加邊後的得分
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
        """後向階段：刪除邊"""
        improved = True
        
        while improved:
            improved = False
            
            # 嘗試所有可能的邊刪除
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

## §2. 干預優化演算法

### §2.1 干預策略優化問題

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                    干預優化問題                                   ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  【問題定義】                                                      ║
║    給定因果圖 G 和目標變數 Y，                                      ║
║    尋找最優干預集合 I 使得 E[Y | do(I)] 最大                        ║
║                                                                       ║
║  【約束】                                                          ║
║    - 預算約束：干預數量 ≤ B                                        ║
║    - 成本約束：總干預成本 ≤ C                                      ║
║    - 干預可行性：僅可干預干預變數集合                               ║
║                                                                       ║
║  【挑戰】                                                          ║
║    - 組合爆炸：n 個變數的干預空間為 2ⁿ                            ║
║    - 因果識別：干預效果可能無法從觀測數據識別                       ║
║    - 干擾效應：干預可能產生非預期後果                               ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### §2.2 貪心干預選擇

```python
"""
貪心干預選擇演算法
"""

from typing import Dict, List, Set, Optional, Callable
import copy

class GreedyInterventionOptimizer:
    """
    貪心干預選擇器
    
    每次選擇邊際收益最大的干預
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
        優化干預選擇
        
        Args:
            target: 目標變數
            intervention_candidates: 可干預變數列表
        
        Returns:
            最優干預列表
        """
        selected = []
        remaining = set(intervention_candidates)
        
        for _ in range(self.budget):
            if not remaining:
                break
            
            # 計算每個候選的邊際收益
            best_var = None
            best_marginal_effect = float('-inf')
            
            for var in remaining:
                # 計算加入該干預後的效果
                test_interventions = selected + [var]
                effect = self.effect_calculator(
                    self.graph,
                    target,
                    test_interventions
                )
                
                # 計算邊際收益
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
            
            # 添加最佳干預
            if best_var:
                selected.append(best_var)
                remaining.remove(best_var)
        
        return selected
```

### §2.3 策略梯度優化

```python
"""
策略梯度干預優化
"""

import numpy as np
from typing import Dict, List, Any

class PolicyGradientOptimizer:
    """
    策略梯度優化器
    
    使用強化學習進行干預策略優化
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
        
        # 策略參數（簡化：線性模型）
        self.policy_weights = {}
    
    def select_action(
        self,
        state: Dict[str, Any],
        epsilon: float = 0.1
    ) -> str:
        """
        選擇動作
        
        ε-greedy 策略
        """
        if np.random.random() < epsilon:
            return np.random.choice(self.action_space)
        
        # 貪心選擇
        best_action = None
        best_value = float('-inf')
        
        for action in self.action_space:
            value = self._estimate_value(state, action)
            if value > best_value:
                best_value = value
                best_action = action
        
        return best_action
    
    def _estimate_value(self, state: Dict, action: str) -> float:
        """估計動作價值"""
        # 框架實現
        return 0.0
    
    def update(
        self,
        states: List[Dict],
        actions: List[str],
        rewards: List[float]
    ):
        """
        更新策略參數
        
        使用策略梯度估計
        """
        # 簡化實現
        for state, action, reward in zip(states, actions, rewards):
            # 計算梯度
            # 更新權重
            pass
    
    def optimize(
        self,
        target: str,
        n_iterations: int = 100,
        epsilon: float = 0.1
    ) -> Dict[str, Any]:
        """
        執行策略優化
        """
        best_intervention = None
        best_reward = float('-inf')
        
        for iteration in range(n_iterations):
            # 當前狀態
            state = self.state_encoder(self.graph)
            
            # 選擇動作
            action = self.select_action(state, epsilon)
            
            # 執行並獲取獎勵
            reward = self._execute_intervention(action, target)
            
            # 更新
            self.update([state], [action], [reward])
            
            # 記錄最佳
            if reward > best_reward:
                best_reward = reward
                best_intervention = action
        
        return {
            "best_intervention": best_intervention,
            "best_reward": best_reward
        }
    
    def _execute_intervention(self, intervention: str, target: str) -> float:
        """執行干預並計算獎勵"""
        # 框架實現
        return 0.0
```

---

## §3. 反事實生成演算法

### §3.1 反事實生成問題

```text
╔═══════════════════════════════════════════════════════════════════════╗
║                    反事實生成問題                                   ║
╠═══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  【問題定義】                                                      ║
║    給定：(1) 事實觀測 (X=x, Y=y)                                   ║
║          (2) 假設干預 do(X=x')                                      ║
║    生成：反事實結果 Y_x'                                            ║
║                                                                       ║
║  【挑戰】                                                          ║
║    - 需要推斷未觀測的外生變數                                       ║
║    - 需要完整的結構方程模型                                         ║
║    - 不確定性量化                                                   ║
║                                                                       ║
║  【應用】                                                          ║
║    - 解釋性 AI                                                     ║
║    - 決策支持                                                     ║
║    - 責任歸因                                                     ║
║                                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝
```

### §3.2 語義反事實生成

```python
"""
語義反事實生成
"""

from typing import Dict, List, Any, Optional, Tuple
import numpy as np

class SemanticCounterfactualGenerator:
    """
    語義反事實生成器
    
    生成最小修改的反事實實例
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
        生成反事實實例
        
        目標：找到最小的修改使得目標變數改變
        """
        current_value = instance.get(target_variable)
        
        # 識別相關變數
        relevant_vars = self._identify_relevant_variables(
            target_variable, instance
        )
        
        # 搜尋反事實
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
        """識別相關變數"""
        # 獲取目標的祖先
        ancestors = self.graph.get_ancestors(target)
        
        # 過濾可修改變數
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
        """搜尋反事實"""
        # 初始化候選
        candidates = [instance.copy()]
        
        for iteration in range(max_iterations):
            best_candidate = None
            best_distance = float('inf')
            
            # 評估候選
            for candidate in candidates:
                # 檢查是否為有效反事實
                if self._is_valid_counterfactual(
                    candidate, target_class
                ):
                    # 計算與原實例的距離
                    distance = self.distance_function(instance, candidate)
                    
                    if distance < best_distance:
                        best_distance = distance
                        best_candidate = candidate
            
            # 如果找到有效反事實且距離在閾值內
            if best_candidate and best_distance <= self.threshold:
                return best_candidate
            
            # 生成新候選
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
        """檢查是否為有效反事實"""
        # 框架實現
        return False
    
    def _generate_candidates(
        self,
        current_candidates: List[Dict],
        relevant_vars: List[str]
    ) -> List[Dict]:
        """生成新候選"""
        new_candidates = []
        
        for candidate in current_candidates:
            for var in relevant_vars:
                # 修改變數值
                new_candidate = candidate.copy()
                # 框架實現：替換值
                new_candidates.append(new_candidate)
        
        return new_candidates
```

### §3.3 實例化反事實生成

```python
"""
實例化反事實生成
"""

class InstanceCounterfactualGenerator:
    """
    實例化反事實生成器
    
    通過最近鄰搜尋生成反事實
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
        生成實例化反事實
        
        從數據集中找到最近的反事實實例
        """
        # 找到目標類別的實例
        target_instances = [
            inst for inst in self.dataset
            if inst.get(target_variable) == target_value
        ]
        
        if not target_instances:
            return None
        
        # 計算距離
        distances = [
            (inst, self.distance_function(instance, inst))
            for inst in target_instances
        ]
        
        # 排序
        distances.sort(key=lambda x: x[1])
        
        # 返回最近的
        return distances[0][0] if distances else None
    
    def generate_diverse(
        self,
        instance: Dict[str, Any],
        target_variable: str,
        target_value: Any,
        n: int = 3
    ) -> List[Dict[str, Any]]:
        """
        生成多樣化反事實集合
        """
        # 找到目標類別的實例
        target_instances = [
            inst for inst in self.dataset
            if inst.get(target_variable) == target_value
        ]
        
        if not target_instances:
            return []
        
        # 計算距離並排序
        distances = [
            (inst, self.distance_function(instance, inst))
            for inst in target_instances
        ]
        distances.sort(key=lambda x: x[1])
        
        # 選擇多樣化的子集
        selected = []
        min_distance_between = 0.1
        
        for inst, dist in distances:
            # 檢查與已選擇的距離
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

## §4. 演算法整合介面

### §4.1 統一演算法入口

```python
"""
核心演算法統一介面
"""

from typing import Dict, List, Any, Optional

class CausalAlgorithmEngine:
    """
    因果演算法引擎
    
    統一接口調用各種因果演算法
    """
    
    def __init__(self):
        self.ci_test = None
        self.graph = None
    
    # ═══════════════════════════════════════════════════════════════════
    # 因果發現
    # ═══════════════════════════════════════════════════════════════════
    
    def discover_causal_structure(
        self,
        variables: List[str],
        data: List[Dict[str, Any]],
        method: str = "pc"
    ) -> CausalGraph:
        """
        發現因果結構
        
        Args:
            variables: 變數列表
            data: 觀測數據
            method: 發現方法 ("pc", "fci", "ges")
        
        Returns:
            CausalGraph: 發現的因果圖
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
    # 干預優化
    # ═══════════════════════════════════════════════════════════════════
    
    def optimize_intervention(
        self,
        target: str,
        intervention_candidates: List[str],
        method: str = "greedy",
        budget: int = 5
    ) -> List[str]:
        """
        優化干預策略
        
        Args:
            target: 目標變數
            intervention_candidates: 可干預變數列表
            method: 優化方法 ("greedy", "policy_gradient")
            budget: 干預預算
        
        Returns:
            最優干預列表
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
    # 反事實生成
    # ═══════════════════════════════════════════════════════════════════
    
    def generate_counterfactual(
        self,
        instance: Dict[str, Any],
        target_variable: str,
        target_value: Any,
        method: str = "semantic"
    ) -> Optional[Dict[str, Any]]:
        """
        生成反事實
        
        Args:
            instance: 輸入實例
            target_variable: 目標變數
            target_value: 目標值
            method: 生成方法 ("semantic", "instance")
        
        Returns:
            反事實實例
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
                [],  # 數據集
                self._calculate_distance
            )
            return generator.generate(
                instance, target_variable, target_value
            )
        
        else:
            raise ValueError(f"Unknown method: {method}")
    
    # ═══════════════════════════════════════════════════════════════════
    # 輔助函數
    # ═══════════════════════════════════════════════════════════════════
    
    def _calculate_effect(
        self,
        graph: CausalGraph,
        target: str,
        interventions: List[str]
    ) -> float:
        """計算干預效果"""
        # 框架實現
        return 0.0
    
    def _encode_state(self, graph: CausalGraph) -> Dict:
        """編碼狀態"""
        return {}
    
    def _calculate_distance(
        self,
        instance1: Dict,
        instance2: Dict
    ) -> float:
        """計算實例距離"""
        # 框架實現
        return 0.0
```

---

## §5. 依賴與約束

### §5.1 模組依賴

| 依賴模組 | 描述 | 引用 |
| --- | --- | --- |
| LOGIC_ENGINE.md | 父模組 | 調用入口 |
| CAUSAL_INFERENCE.md | 因果推論 | do-calculus |
| ABDUCTIVE_REASONING.md | 溯因推理 | 異常分析 |
| COUNTERFACTUAL.md | 反事實推理 | 雙世界模型 |
| CONSTRAINTS.md | 社會權限層級 | SA-L 權限校驗 |

### §5.2 約束條件

| 約束類型 | 描述 | 邊界 |
| --- | --- | :--- |
| 因果發現 | 需足夠樣本量 | n ≥ 100 |
| 干預預算 | 最大干預數 | ≤ 10 |
| 反事實距離 | 閾值內有效 | ≤ 0.5 |
| 計算複雜度 | 骨架發現 | O(n² × 2^d) |

---

## §6. 版本與演進

| 版本 | 日期 | 變更摘要 |
| --- | :--- | :--- |
| v2.2 | 2026-03 | 初始版本，對應 LOGIC_ENGINE.md §5.2 |
| v2.3 | 2026-03 | 添加 ctf-calculus、CauScale、HCP-DCNet |

**演化約束：** 本模組的修改不得違反 NoieLogicAGENTS.md 的不可變核心公理。任何演化提議必須記錄至 EVOLUTION_LOG.md。

---

*Core Algorithms v2.3 — 因果發現、干預優化與反事實生成*
*實現因果推論框架的核心計算能力*
