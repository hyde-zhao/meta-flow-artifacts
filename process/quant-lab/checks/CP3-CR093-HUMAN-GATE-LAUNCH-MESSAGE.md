请审查：process/checkpoints/CP3-CR093-LEDGER-HYGIENE-HLD-REVIEW.md

自动预检结论：PASS

Context Capsule：process/context/CP3-CR093-LEDGER-HYGIENE-CONTEXT.yaml（read_profile=compact，完整来源见 checklist）

本轮待人工决策项：4

决策收集覆盖：已扫描 6 个来源，发现候选问题 22 个，纳入待决策 4 个；N/A 来源 0 个，原因见 checklist 的 Decision Collection Coverage。

如果你回复 approve，表示你接受以下 4 项推荐方案，不表示授权以下 8 项禁止操作。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP3-CR093-01 | architecture | checker 当前状态读取模型如何冻结？ | 采用 current / tracking-current / audit-history 三层模型；history 不作为 current failure。 | 继续全文扫描；完全跳过 history。 | 推荐方案兼顾审计与准确性；全文扫描继续误报。 | 修复 CR025 / CR092 nested active_change 误报。 |
| DQ-CP3-CR093-02 | implementation | CP5 设计是否采用状态等价表 + 最小 CR019 台账规范化？ | 采用集中状态等价表，并只做必要文档规范化。 | 只改文档；只改 checker；全量重写 checker。 | 推荐方案修根因且改动可控；全量重写成本高。 | CP5 需输出 fixture、回滚和文件 owner。 |
| DQ-CP3-CR093-03 | security | CP3 approve 是否继续不授权实施和外部动作？ | 继续不授权；只允许进入 CP5 LLD / TEST-PLAN / TASKS。 | 直接授权修 checker；授权读取 NAS / runtime / 账户证据。 | 推荐方案符合门禁；直接实施会越过 CP5。 | 防止把架构批准误读成实施批准。 |
| DQ-CP3-CR093-04 | risk_acceptance | broader `source=cp8-follow-up` missing tracking row warnings 是否本轮强修？ | 不强修，保持 warning-only；本轮只治理 CR019 / CR025 阻断项。 | 本轮一起修所有 warnings；全部 suppress。 | 推荐方案控制范围；一起修会扩大到多个旧 CR。 | CP7 后可能仍有 warning。 |

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
