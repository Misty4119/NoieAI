# RETROCAUSAL/ — Retrocausal Knowledge Update Module

---

## BIDIRECTIONAL_PROPAGATION.md

### Bidirectional Belief Propagation

Supports bidirectional belief propagation along the timeline.

```python
FUNCTION BidirectionalPropagation(knowledge_graph, new_evidence):
    # Forward propagation: past → future
    forward_messages = ForwardPropagate(knowledge_graph, new_evidence)
    
    # Retrograde propagation: future → past
    retro_messages = RetroPropagate(knowledge_graph, new_evidence)
    
    # Fuse updates
    return Fuse(forward_messages, retro_messages)
```

---

## RETRO_ENTANGLEMENT.md

### Retroactive Entanglement Pointer Management

Records knowledge dependencies on future premises that may be corrected.

```python
FUNCTION ManageRetroEntanglement(knowledge):
    knowledge.retro_pointer = {
        "dependencies": TraceFutureDependencies(knowledge),
        "update_trigger": SetTrigger(knowledge)
    }
    return knowledge
```
