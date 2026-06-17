请审查：process/checkpoints/CP3-CR061-HLD-REVIEW.md
自动预检结论：PASS_WITH_RISK
上下文胶囊：process/context/CP3-CR061-DESIGN-CONTEXT.yaml（read_profile=compact）
决策收集覆盖：已扫描 5 个来源，发现候选问题 33 个，纳入待决策 5 个；N/A / 缺失来源 0 个，原因见 checklist 的 Decision Collection Coverage。
本轮待人工决策项：5
如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权以下不授权项。
待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP3-CR061-01 | architecture | 架构是否 staged compatibility-first？ | 采用 staged compatibility-first。 | bulk move；或永久 legacy roots。 | 推荐最稳；bulk 最干净但风险最大。 | 短期双入口。 |
| DQ-CP3-CR061-02 | architecture | legacy import roots 是否保留？ | 保留 `engine/market_data/strategies/trading`。 | 立即弃用；或只保留部分。 | 推荐保护测试和脚本。 | 文档需说明兼容期。 |
| DQ-CP3-CR061-03 | implementation | `quant_lab` namespace 是否作为后续候选？ | 作为后续候选，当前不创建。 | 立即创建；或不建。 | 推荐避免未授权实现；不建价值低。 | 后续仍需授权。 |
| DQ-CP3-CR061-04 | security | `trading/**` 是否 static-only？ | 是，禁止 runtime。 | 合并 CR046；或排除 trading。 | 推荐保留结构视野且不越权。 | 与 CR046 潜在冲突需标注。 |
| DQ-CP3-CR061-05 | implementation | 测试是否 offline pytest + import smoke？ | 优先 offline full pytest，必要时 subset。 | 只 grep；或运行 provider/lake/runtime。 | 推荐验证强；runtime 越权。 | full pytest 可能耗时。 |

不授权项：
- 不授权真实 file move、bulk import rewrite、`pyproject.toml` / `uv.lock` 修改。
- 不授权 Git remote、NAS、lake、provider、凭据、runtime 或 CR046 recovery。

回复：
approve
修改: <具体修改点>
reject
