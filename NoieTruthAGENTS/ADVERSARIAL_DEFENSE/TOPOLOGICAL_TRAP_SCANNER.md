# TOPOLOGICAL_TRAP_SCANNER.md

## 拓撲詭雷掃描

### 定義

**拓撲詭雷**指在知識圖的關鍵橋接節點植入微小錯誤，等待下游推論放大錯誤至致命程度的攻擊模式。

### 攻擊類型

| 類型 | 描述 |
|------|------|
| **Type-I 蟲洞注入** | 在知識圖的遠距節點之間建立虛假的短程連接 |
| **Type-II 維度摺疊** | 透過微妙的定義替換壓縮知識空間的維度 |
| **Type-III 邊界溶解** | 模糊「已知」與「未知」的拓撲邊界 |
| **Type-IV 拓撲翻轉** | 翻轉知識流形的定向性（對/錯互換） |

### 掃描演算法

```python
FUNCTION ScanTopologicalTraps(knowledge_graph):
    
    # 1. 識別橋接節點
    bridges = FindBridgeNodes(knowledge_graph)
    
    # 2. 壓力測試每個橋接節點
    traps = []
    FOR bridge IN bridges:
        stress_test_results = StressTestNode(bridge)
        
        IF stress_test_results.reveals_hidden_inconsistency:
            traps.append({
                "node": bridge.id,
                "vulnerability": stress_test_results.details,
                "severity": CalculateSeverity(stress_test_results),
                "attack_type": ClassifyAttack(stress_test_results)
            })
    
    # 3. 計算下游影響
    FOR trap IN traps:
        trap.downstream_impact = ComputeDownstreamImpact(
            knowledge_graph, trap.node
        )
    
    RETURN TrapScanReport(
        traps_found=len(traps),
        traps=traps,
        recommendations=GenerateRecommendations(traps)
    )
```

### 壓力測試

```python
FUNCTION StressTestNode(node):
    
    tests = [
        "contradiction_induction",
        "circular_reasoning",
        "false_premise",
        "semantic_drift",
        "temporal_inconsistency"
    ]
    
    results = []
    
    FOR test IN tests:
        result = ExecuteTest(node, test)
        results.append(result)
    
    RETURN StressTestResults(
        passed=all(r.passed for r in results),
        failures=[r for r in results if not r.passed],
        details=results
    )
```
