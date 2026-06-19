请审查：process/checkpoints/CP3-CR092-READONLY-RUNTIME-SMOKE-HLD-REVIEW.md

自动预检结论：PASS

Context Capsule：process/context/CP3-CR092-DESIGN-CONTEXT.yaml（read_profile=compact，完整来源见 checklist）

本轮待人工决策项：4

决策收集覆盖：已扫描 6 个来源，发现候选问题 17 个，纳入待决策 4 个；N/A / 缺失来源 0 个，原因见 checklist 的 Decision Collection Coverage。

如果你回复 approve，表示你接受以下 4 项推荐方案，不表示授权以下 7 项禁止操作。用户已说明当前为模拟账户测试，DQ-CP3-CR092-02 的推荐方案已调整为允许读取用户明确提供的模拟账户测试证据；这不授权读取凭据、真实账户、NAS、未指定原始日志或启动 runtime。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP3-CR092-01 | architecture | CR092 HLD 是否采用人工逐 run 只读 smoke 设计门禁？ | 采用方案 A，CP5 前只冻结授权、manual guide、evidence 合同。 | runner 内置真实 readonly transport；合并 CR089；暂停 CR092。 | 推荐方案权限最小；备选自动化更强但风险高。 | 真实 runtime readiness 仍未证明。 |
| DQ-CP3-CR092-02 | security | 是否允许读取模拟账户测试证据？ | 允许读取用户明确提供路径 / 内容的模拟账户测试证据；优先脱敏 summary，也可读取模拟账户的持仓 / 委托 / 成交 / 日志片段；禁止 `.env`、凭据、真实账户、真实资金 / 持仓 / 委托 / 成交、未指定日志和 NAS。 | 仍只读脱敏 summary；或扩大到真实账户原文和完整日志。 | 推荐方案贴合模拟账户测试且边界清楚；脱敏-only 更安全但信息可能不足；真实账户 / 完整日志风险过高。 | 需要用户给出明确文件路径或粘贴内容；发现凭据、真实账户、live 标识或 NAS 路径则停止。 |
| DQ-CP3-CR092-03 | runtime_authorization | CP3 approve 是否授权真实运行？ | 不授权；只允许进入 CP5 LLD / TEST-PLAN / readiness。 | 同时授权一次只读 smoke；授权实现真实 transport。 | 推荐方案符合 CP2；备选需要独立逐 run 授权。 | 防止设计通过被误读成 runtime approve。 |
| DQ-CP3-CR092-04 | follow_up_tracking | NAS、order-write、ledger hygiene 如何处理？ | 继续保留 CR091-FU-02、FU-03、FU-04，CR092 不合并。 | 本轮合并 NAS；本轮合并 order-write；先暂停 CR092 修 ledger。 | 推荐方案不扩大范围；备选会引入新风险或打断主线。 | cr-tracking 旧账仍会非阻断存在。 |

不授权项：

- QMT / MiniQMT / XtQuant / gateway / runner runtime 启动、连接、安装或运行。
- NAS 访问、列取、读取、复制、拉取、写入、发布或删除。
- 读取 `.env`、凭据、真实账号、真实账户、真实资金 / 持仓 / 委托 / 成交、未指定日志原文；但允许读取用户明确提供的模拟账户测试证据。
- submit / cancel、buy / sell。
- simulation / live。
- provider fetch、lake write、catalog publish。
- 自动启动 CR089 或恢复 CR020 用户删除的 gateway 路线。

请回复以下三者之一：

- `approve`
- `修改: <具体修改点>`
- `reject`
