请审查：process/checkpoints/CP2-CR093-LEDGER-HYGIENE-SCOPE-REVIEW.md

自动预检结论：PASS

Context Capsule：process/context/CP2-CR093-LEDGER-HYGIENE-CONTEXT.yaml（read_profile=compact，完整来源见 checklist）

本轮待人工决策项：5

决策收集覆盖：已扫描 7 个来源，发现候选问题 14 个，纳入待决策 5 个；N/A 来源 1 个，原因见 checklist 的 Decision Collection Coverage。

如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权以下 8 项禁止操作。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP2-CR093-01 | scope | CR093 是否只治理 CR019 / CR025 相关历史账本与 `cr-tracking` 一致性？ | 是，仅覆盖静态账本、状态索引和检查器语义。 | 扩大到 runtime / NAS 台账；取消 CR093 保留旧账。 | 推荐方案聚焦目标缺陷且风险低；扩大范围会混入高风险授权。 | 不处理交易业务或真实环境。 |
| DQ-CP2-CR093-02 | implementation | 后续修复路线采用什么策略？ | 采用“检查器语义修正 + 最小台账规范化”。 | 只改旧账本迎合旧检查器；只 suppress 警告；重写全量 checker。 | 推荐方案能修根因且控制改动面；其他方案要么保留根因要么成本过高。 | CP5 后可能修改 checker、测试 fixture、CR019 follow-up tracking 和状态索引。 |
| DQ-CP2-CR093-03 | security | CR093 是否继续禁止 runtime、NAS、凭据、账户、交易写和数据发布？ | 继续全部禁止。 | 允许读取 NAS 台账；允许执行 runtime smoke；允许读取账户证据。 | 推荐方案符合 ledger-only 范围；备选会引入外部状态和敏感边界。 | 无外部副作用。 |
| DQ-CP2-CR093-04 | risk_acceptance | 是否接受历史正文保留但 checker 不把历史审计文本当作当前缺陷？ | 接受；历史追溯保留，当前状态以权威字段 / 当前表格为准。 | 删除历史片段；继续全量文本扫描；把所有旧状态统一改写。 | 推荐方案兼顾追溯与检查准确性；删除历史会损失审计。 | 需要明确“当前状态”和“历史审计文本”的边界。 |
| DQ-CP2-CR093-05 | follow_up_tracking | CR092-FU-02 runtime、CR091-FU-02 NAS、CR091-FU-03 order-write 如何处理？ | 全部保持候选，不随 CR093 启动。 | 合并 runtime / NAS / order-write；取消全部候选。 | 推荐方案避免范围膨胀且保留后续路线。 | CR093 approve 只推进 ledger hygiene。 |

不授权项：

- 修改 `scripts/check_cr_tracking_consistency.py` 或旧账本正文。
- QMT / MiniQMT / XtQuant / gateway / runner runtime 启动、连接、安装或运行。
- NAS 访问、列取、读取、复制、拉取、写入、发布或删除。
- 读取 `.env`、凭据、账号、账户、资金、持仓、委托、成交或日志原文。
- submit / cancel、buy / sell。
- simulation / live。
- provider fetch、lake write、catalog publish。
- 自动启动 CR089、CR091-FU-02、CR091-FU-03 或恢复 CR020 用户删除的 gateway 路线。

请回复以下三者之一：

- `approve`
- `修改: <具体修改点>`
- `reject`
