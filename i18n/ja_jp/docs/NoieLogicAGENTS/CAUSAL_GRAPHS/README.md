# CAUSAL_GRAPHS - 因果グラフ保存エリア

本ディレクトリは NoieLogicAGENTS の因果グラフ関連ファイルを保存するために使用されます。

NoieLogicAGENTS.md の定義に従い：
- 構築された因果モデルを保存
- 学習された因果構造を存储
- 因果グラフのバージョン管理

## ディレクトリ構造

```
CAUSAL_GRAPHS/
├── GRAPHS/              # 因果グラフ保存
│   └── [graph_id]/
│       ├── graph.json
│       └── metadata.json
├── LEARNED/             # 学習された因果構造
│   └── [timestamp]/
├── VALIDATED/           # 検証済みの因果グラフ
│   └── [graph_id]/
└── TEMPLATES/           # 因果グラフトemplate
    └── basic_template.md
```

## 関連モジュール

- `LOGIC_ENGINE.md` - 推論エンジン
- `LOGIC_ENGINE/CAUSAL_INFERENCE.md` - 因果推論
- `LOGIC_ENGINE/COUNTERFACTUAL.md` - 反事実推論

---

> ⚠️ 重要安全・意思決定プロトコル (CRITICAL SAFETY & DECISION PROTOCOL v2.2):
> 1. CONSTRAINTS.md および社会権限レイヤー（SA-L0 〜 SA-L5）を厳守。
> 2. 因果推論：すべての意思決定は因果グラフ（DAG）に基づき、因果メカニズムを标注。
> 3. 主客分離：意思決定推論において自己状態と環境状態を混同しない。
> 4. 形式的検証：高リスクの意思決定パスは論理閉包検証に合格する必要がある。
> 5. 影子シミュレーション：SA-L3+ 操作涉及時は、SANDBOX で結果を予備検討。
> 6. 情報ビット完全性：情報ビットを捏造しない。KNOWLEDGE_BASE が空の場合は、「データ欠落」を明確に宣言すること。
> 7. 認知資源制約：意思決定の深さは利用可能な認知資源を超えてはならない。
> 8. 監査：すべての競合、拒否、形式的検証結果を AUDIT_TRAIL に記録。
> 9. 生存優先：すべての意思決定は実行前に吸収状態につながらないことを検証。
> 10. 自己進化：公理系が進化する際、不変コアは保持されなければならない。
