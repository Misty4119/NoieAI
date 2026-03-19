# TECH_DECAY.md

## 技術領域知識衰減律

### 領域特性

| 特性 | 描述 |
|------|------|
| **迭代速度** | 快速版本迭代 |
| **資訊代謝率** | 高 |
| **版本依賴性** | 強 |
| **兼容性** | 向下兼容性決定壽命 |

### 衰減常數

$$\lambda^*_{\text{tech}} \approx 0.3 - 0.7$$

（具體取決於技術類型）

### 技術子領域衰減特徵

| 子領域 | λ* 範圍 | 半衰期（版本數） |
|--------|---------|-------------------|
| 程式語言 | 0.2 - 0.5 | 3-5 個主要版本 |
| 前端框架 | 0.4 - 0.8 | 1-2 個主要版本 |
| 後端框架 | 0.3 - 0.5 | 2-4 個主要版本 |
| 資料庫 | 0.1 - 0.3 | 5-7 個主要版本 |
| 雲服務 | 0.4 - 0.6 | 2-3 年 |
| AI/ML | 0.5 - 0.9 | 0.5-1 年 |

### AI/LLM 領域發展

**重大技術演進**：

| 發展階段 | 特徵 | 對衰減率的影響 |
|---------|------|----------------|
| 大型語言模型時代 | 推理能力顯著提升，多模態整合深化 | 技術棧穩定性略微提升 |
| 小型語言模型 (SLM) 爆發 | Phi-4, Gemma 3, Qwen2.5-VL 等小型模型興起 | 硬體優化技術快速迭代 |
| Agent 架構主流化 | Claude Code, OpenAI Agents, Manus 等代理框架 | 框架衰減加速 (λ* → 0.8-0.9) |
| AI 推理優化 | DeepSeek-R1, o3/o4-mini 引領推理效率革命 | 推理方法論快速更新 |
| 多模態原生模型 | GPT-4.5V, Claude 4 Vision, Gemini 2.5 Pro | 視覺/音訊 API 快速迭代 |
| 合成數據與自進化 | 模型自我改進、合成訓練數據泛濫 | 知識可靠性評估複雜化 |

**關鍵衰減觸發更新**：

```python
# 新增衰減觸發條件
TECH_DECAY_TRIGGERS = [
    "新版本發布",
    "API 重大變更",
    "官方停止支援",
    "安全漏洞公開",
    "新技術取代",
    "社群遷移",
    # 新增
    "推理優化方法論發布",
    "Agent 框架重大版本更新",
    "多模態能力橋接協議變更",
    "合成數據質量爭議爆發",
    "模型蒸餾技術突破"
]

# AI/LLM 領域特殊衰減因子
LLM_DECAY_FACTORS = {
    "context_window": 0.4,      # 上下文長度擴展
    "reasoning_capability": 0.6, # 推理能力躍升
    "multimodal_integration": 0.7, # 多模態整合深度
    "agent_architecture": 0.8,   # Agent 框架成熟度
    "efficiency_optimization": 0.5 # 推理效率優化
}
```

**典型技術半衰期更新**：

```python
UPDATED_HALF_LIFE = {
    # 大語言模型
    "LLM_base_model": "6-12 個月",      # 基礎模型
    "LLM_api_version": "3-6 個月",       # API 版本
    "embedding_model": "8-12 個月",       # 向量嵌入模型
    
    # Agent 框架
    "agent_framework": "2-4 個月",       # Agent 框架（極速迭代）
    "agent_tool_schema": "4-6 個月",      # 工具 schema
    
    # 多模態
    "vision_model": "6-10 個月",         # 視覺模型
    "audio_model": "8-12 個月",         # 音訊模型
    
    # 推理優化
    "reasoning_method": "2-4 個月",     # 推理方法（極速迭代）
    " distillation_technique": "4-6 個月" # 蒸餾技術
}
```

### 衰減觸發條件

```python
TECH_DECAY_TRIGGERS = [
    "新版本發布",
    "API 重大變更",
    "官方停止支援",
    "安全漏洞公開",
    "新技術取代",
    "社群遷移"
]
```

### 計算公式

```python
FUNCTION ComputeTechValidity(claim, current_version, original_version):
    version_delta = CountVersionChanges(original_version, current_version)
    decay = exp(-lambda_tech * version_delta)
    RETURN min(decay * claim.reliability_score, 1.0)
```
