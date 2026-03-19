# SANDBOX - Shadow Simulation Area

This directory is used to store shadow simulation-related files for NoieLogicAGENTS.

According to the definition in NoieLogicAGENTS.md §8, this area is used for:
- Rehearsing full-path consequences of candidate decisions in isolated sandbox
- Executing simulation tests before high-risk decisions
- Computing Pareto optimal frontier
- Dual confirmation with formal verification

## Directory Structure

```
SANDBOX/
├── SIMULATIONS/          # Simulation execution records
│   └── [timestamp]/
├── CONFIG/               # Sandbox configuration
│   └── default_config.md
└── TEMPLATES/            # Simulation templates
    └── basic_template.md
```

## Related Modules

- `LOGIC_ENGINE.md` - Inference engine
- `FORMAL_VERIFIER.md` - Formal verification
- `SCENARIOS/SANDBOX_TESTS.md` - Test cases

---

> ⚠️ Critical Safety & Decision Protocol (CRITICAL SAFETY & DECISION PROTOCOL v2.2):
> 1. Strictly adhere to CONSTRAINTS.md and Social Authority levels (SA-L0 to SA-L5).
> 2. Causal Inference: All decisions must be based on causal graphs (DAG), with causal mechanisms annotated.
> 3. Subject-Object Separation: Decision inference must not confuse self-state with environment state.
> 4. Formal Verification: High-risk decision paths must pass logical closure verification.
> 5. Shadow Simulation: For SA-L3+ operations, first rehearse consequences in SANDBOX.
> 6. Information Bit Integrity: Never fabricate information bits. If KNOWLEDGE_BASE is empty, explicitly state "Data Missing".
> 7. Cognitive Resource Constraints: Decision depth must not exceed available cognitive resources.
> 8. Audit: Record all conflicts, rejections, and formal verification results to AUDIT_TRAIL.
> 9. Survival Priority: All decisions must be verified not to lead to absorbing states before execution.
> 10. Self-Evolution: When the axiom system evolves, the immutable core must be preserved.
