# RETROCAUSAL/ — 逆因果知識更新モジュール

---

## BIDIRECTIONAL_PROPAGATION.md

### 双方向信念伝播

時間軸双方向の信念伝播をサポート。

```python
FUNCTION BidirectionalPropagation(knowledge_graph, new_evidence):
    # 前方伝播：過去 → 未来
    forward_messages = ForwardPropagate(knowledge_graph, new_evidence)
    
    # 逆方伝播：未来 → 過去
    retro_messages = RetroPropagate(knowledge_graph, new_evidence)
    
    # 融合更新
    return Fuse(forward_messages, retro_messages)
```

---
