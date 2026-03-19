# RETROCAUSAL/ — 逆因果知識更新模組

---

## BIDIRECTIONAL_PROPAGATION.md

### 雙向信念傳播

支援時間軸雙向的信念傳播。

```python
FUNCTION BidirectionalPropagation(knowledge_graph, new_evidence):
    # 前向傳播：過去 → 未來
    forward_messages = ForwardPropagate(knowledge_graph, new_evidence)
    
    # 逆向傳播：未來 → 過去
    retro_messages = RetroPropagate(knowledge_graph, new_evidence)
    
    # 融合更新
    return Fuse(forward_messages, retro_messages)
```

---

## RETRO_ENTANGLEMENT.md

### 逆時間糾纏指針管理

記錄知識依賴哪些未來可能被修正的前提。

```python
FUNCTION ManageRetroEntanglement(knowledge):
    knowledge.retro_pointer = {
        "dependencies": TraceFutureDependencies(knowledge),
        "update_trigger": SetTrigger(knowledge)
    }
    return knowledge
```
