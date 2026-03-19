# SANDBOX - 影子模擬專區

此目錄用於存放 NoieLogicAGENTS 的影子模擬相關檔案。

根據 NoieLogicAGENTS.md §8 的定義，此區域用於：
- 在隔離沙盒中預演候選決策的全路徑後果
- 執行高風險決策前的模擬測試
- 計算帕累托最優前沿
- 與形式化驗證進行雙重確認

## 目錄結構

```
SANDBOX/
├── SIMULATIONS/          # 模擬執行記錄
│   └── [timestamp]/
├── CONFIG/               # 沙盒配置
│   └── default_config.md
└── TEMPLATES/            # 模擬模板
    └── basic_template.md
```

## 相關模組

- `LOGIC_ENGINE.md` - 推理引擎
- `FORMAL_VERIFIER.md` - 形式化驗證
- `SCENARIOS/SANDBOX_TESTS.md` - 測試案例

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
