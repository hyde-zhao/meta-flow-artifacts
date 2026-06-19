请审查：process/checkpoints/CP5-CR092-READONLY-RUNTIME-SMOKE-READINESS.md

自动预检结论：PASS

Context Capsule：process/context/CP5-CR092-READINESS-CONTEXT.yaml（read_profile=compact，完整来源见 checklist）

本轮待人工决策项：6

决策收集覆盖：已扫描 6 个来源，发现候选问题 19 个，纳入待决策 6 个；N/A / 缺失来源 0 个，原因见 checklist 的 Decision Collection Coverage。

如果你回复 approve，表示你接受以下 6 项推荐方案，不表示授权以下 7 项禁止操作。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP5-CR092-01 | implementation | 是否批准 CR092 LLD 作为 readiness 设计证据？ | 批准当前 LLD，允许进入 manual guide / evidence template 准备。 | 修改 LLD 后重提；reject 回 CP3 approved / CP5 redesign。 | 推荐方案完整；修改更精细但延后。 | 设计通过不等于运行通过。 |
| DQ-CP5-CR092-02 | security | 是否批准模拟账户 evidence 合同？ | 允许读取用户明确提供的模拟账户测试证据；拒收凭据、真实账户、NAS 和未指定日志。 | 仅允许脱敏 summary；或扩大到真实账户原文。 | 推荐方案兼顾验证和边界；真实账户原文风险过高。 | 证据混入敏感信息时必须停止读取。 |
| DQ-CP5-CR092-03 | runtime_authorization | CP5 approve 是否授权真实运行？ | 不授权；只允许准备 manual guide / evidence template / 静态 checker 范围。 | 同时授权一次只读 smoke；授权 runner 自动 transport。 | 推荐方案符合 CP2/CP3；备选需要独立逐 run 授权。 | 防止 readiness 被误读为 runtime approve。 |
| DQ-CP5-CR092-04 | follow_up_tracking | NAS、order-write、ledger hygiene 是否仍独立分流？ | 保持 CR091-FU-02、FU-03、FU-04 独立，不并入 CR092。 | 本轮合并 NAS；本轮合并 order-write；先暂停 CR092 修 ledger。 | 推荐方案不扩大范围；备选会引入新风险。 | cr-tracking 旧账仍非阻断存在。 |
| DQ-CP5-CR092-05 | risk_acceptance | 是否接受真实 runtime readiness 仍未被证明？ | 接受，当前交付是 readiness 设计证据，后续需要 CP6/CP7 或逐 run evidence 才能证明。 | 要求现在证明真实 runtime；取消 CR092。 | 推荐方案诚实表达剩余风险；立即证明需要新授权。 | 不能把 CP5 作为成功 smoke 证据。 |
| DQ-CP5-CR092-06 | implementation | CP5 approve 后下一步允许做什么？ | 只允许准备 manual guide、evidence template 和可选静态 checker 范围，不执行 runtime。 | 直接执行 smoke；直接实现 runner transport。 | 推荐方案可控；备选越过授权。 | 后续实现仍需 CP6 / CP7 或独立 runtime gate。 |

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
