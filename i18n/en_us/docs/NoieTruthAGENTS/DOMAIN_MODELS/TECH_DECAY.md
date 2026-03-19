# TECH_DECAY.md

## Technology Domain Knowledge Decay Law

### Domain Characteristics

| Feature | Description |
|---------|-------------|
| **Iteration Speed** | Rapid version iterations |
| **Information Metabolism Rate** | High |
| **Version Dependency** | Strong |
| **Compatibility** | Backward compatibility determines lifespan |

### Decay Constants

$$\lambda^*_{\text{tech}} \approx 0.3 - 0.7$$

(Specific values depend on technology type)

### Technology Sub-Domain Decay Characteristics

| Sub-Domain | λ* Range | Half-Life (Version Count) |
|-----------|-----------|---------------------------|
| Programming Languages | 0.2 - 0.5 | 3-5 major versions |
| Frontend Frameworks | 0.4 - 0.8 | 1-2 major versions |
| Backend Frameworks | 0.3 - 0.5 | 2-4 major versions |
| Databases | 0.1 - 0.3 | 5-7 major versions |
| Cloud Services | 0.4 - 0.6 | 2-3 years |
| AI/ML | 0.5 - 0.9 | 0.5-1 years |

### AI/LLM Domain Developments

**Major Technology Evolution**:

| Development Phase | Characteristics | Impact on Decay Rate |
|------------------|----------------|---------------------|
| Large Language Model Era | Significantly improved reasoning capabilities, deeper multimodal integration | Technology stack stability slightly improved |
| Small Language Model (SLM) Explosion | Rise of small models like Phi-4, Gemma 3, Qwen2.5-VL | Hardware optimization technology rapidly iterates |
| Agent Architecture Mainstreaming | Agent frameworks like Claude Code, OpenAI Agents, Manus | Framework decay accelerates (λ* → 0.8-0.9) |
| AI Reasoning Optimization | DeepSeek-R1, o3/o4-mini lead reasoning efficiency revolution | Reasoning methodologies rapidly update |
| Multimodal Native Models | GPT-4.5V, Claude 4 Vision, Gemini 2.5 Pro | Vision/audio APIs rapidly iterate |
| Synthetic Data and Self-Evolution | Model self-improvement, synthetic training data proliferation | Knowledge reliability evaluation becomes more complex |

**Key Decay Trigger Updates**:

```python
# New decay trigger conditions
TECH_DECAY_TRIGGERS = [
    "New version releases",
    "Major API changes",
    "Official support discontinued",
    "Security vulnerabilities disclosed",
    "New technology replaces old",
    "Community migration",
    # New additions
    "Reasoning optimization methodology published",
    "Agent framework major version updates",
    "Multimodal capability bridging protocol changes",
    "Synthetic data quality controversy erupts",
    "Model distillation technology breakthroughs"
]

# AI/LLM Domain Special Decay Factors
LLM_DECAY_FACTORS = {
    "context_window": 0.4,      # Context length expansion
    "reasoning_capability": 0.6, # Reasoning capability leaps
    "multimodal_integration": 0.7, # Multimodal integration depth
    "agent_architecture": 0.8,   # Agent framework maturity
    "efficiency_optimization": 0.5 # Reasoning efficiency optimization
}
```

**Typical Technology Half-Life Updates**:

```python
UPDATED_HALF_LIFE = {
    # Large Language Models
    "LLM_base_model": "6-12 months",      # Base models
    "LLM_api_version": "3-6 months",       # API versions
    "embedding_model": "8-12 months",       # Vector embedding models
    
    # Agent Frameworks
    "agent_framework": "2-4 months",       # Agent frameworks (extremely rapid iteration)
    "agent_tool_schema": "4-6 months",      # Tool schemas
    
    # Multimodal
    "vision_model": "6-10 months",         # Vision models
    "audio_model": "8-12 months",         # Audio models
    
    # Reasoning Optimization
    "reasoning_method": "2-4 months",     # Reasoning methods (extremely rapid iteration)
    "distillation_technique": "4-6 months" # Distillation techniques
}
```

### Decay Trigger Conditions

```python
TECH_DECAY_TRIGGERS = [
    "New version releases",
    "Major API changes",
    "Official support discontinued",
    "Security vulnerabilities disclosed",
    "New technology replaces old",
    "Community migration"
]
```

### Calculation Formula

```python
FUNCTION ComputeTechValidity(claim, current_version, original_version):
    version_delta = CountVersionChanges(original_version, current_version)
    decay = exp(-lambda_tech * version_delta)
    RETURN min(decay * claim.reliability_score, 1.0)
```
