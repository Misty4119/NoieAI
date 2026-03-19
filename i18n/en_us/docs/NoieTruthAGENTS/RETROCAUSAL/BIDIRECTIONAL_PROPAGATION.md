# RETROCAUSAL/ — Retrocausal Knowledge Update Module

---

## BIDIRECTIONAL_PROPAGATION.md

### Bidirectional Belief Propagation

Supports time-axis bidirectional belief propagation.

```python
FUNCTION BidirectionalPropagation(knowledge_graph, new_evidence):
    # Forward propagation: past → future
    forward_messages = ForwardPropagate(knowledge_graph, new_evidence)
    
    # Retrograde propagation: future → past
    retro_messages = RetroPropagate(knowledge_graph, new_evidence)
    
    # Fusion update
    return Fuse(forward_messages, retro_messages)
```

---

## RETRO_ENTANGLEMENT.md

### Retro-Time Entanglement Pointer Management

Records which future-revisable premises the knowledge depends on.

```python
FUNCTION ManageRetroEntanglement(knowledge):
    knowledge.retro_pointer = {
        "dependencies": TraceFutureDependencies(knowledge),
        "update_trigger": SetTrigger(knowledge)
    }
    return knowledge
```
