请审查：process/checkpoints/CP2-CR092-READONLY-RUNTIME-SMOKE-SCOPE-REVIEW.md

自动预检结论：PASS

Context Capsule：process/context/CP2-CR092-REQUIREMENT-CONTEXT.yaml（read_profile=compact，完整来源见 checklist）

本轮待人工决策项：5

决策收集覆盖：已扫描 6 个来源，发现候选问题 13 个，纳入待决策 5 个；N/A 来源 1 个，原因见 checklist 的 Decision Collection Coverage。

如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权以下 7 项禁止操作。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP2-CR092-01 | scope | CR092 是否只作为真实只读 runtime smoke 的设计门禁？ | 是，仅确认范围并进入 CP3 / CP5 设计，不执行真实 smoke。 | 直接进入真实 runtime；取消 CR092 回到 follow-up backlog。 | 推荐方案保留推进路径且不越权；直接 runtime 风险过高。 | 真实可用性仍未证明。 |
| DQ-CP2-CR092-02 | runtime_authorization | CP2 approve 是否授权 QMT/MiniQMT/XtQuant/gateway/runner 启动或连接？ | 不授权。 | 授权一次只读 smoke；授权准备真实运行脚本。 | 推荐方案符合当前门禁；备选必须先完成 HLD/LLD 和逐 run 授权。 | 防止把设计批准误读成执行授权。 |
| DQ-CP2-CR092-03 | security | NAS、凭据、账户原文、submit/cancel、simulation/live、provider/lake/publish 是否继续禁止？ | 继续全部禁止。 | 放开只读账户查询；放开 NAS package exchange。 | 推荐方案最小风险；备选会引入凭据 / NAS / 账户边界。 | 后续 HLD 必须用脱敏 evidence 和禁止动作计数。 |
| DQ-CP2-CR092-04 | architecture | CR092 与 CR089 的关系如何处理？ | CR092 独立推进设计门禁，CR089 保持 blocked，不自动启动。 | 合并进 CR089；先恢复 CR089；取消 CR092。 | 推荐方案避免恢复 NAS package exchange 和 CR089 runtime。 | 两者语义相关但授权边界不同。 |
| DQ-CP2-CR092-05 | implementation | CP2 approve 后下一步允许做什么？ | 只允许生成 CP3 HLD、CP5 LLD / TEST-PLAN / readiness，不写真实 runtime 代码，不执行 smoke。 | 允许实现 manual guide；允许实现 runner runtime adapter。 | 推荐方案先设计后实现；备选需要 CP5 后再评估。 | CP3/CP5 未 approved 前不得实施。 |

不授权项：

- QMT / MiniQMT / XtQuant / gateway / runner runtime 启动、连接、安装或运行。
- NAS 访问、列取、读取、复制、拉取、写入、发布或删除。
- 读取 `.env`、凭据、账号、账户、资金、持仓、委托、成交或日志原文。
- submit / cancel、buy / sell。
- simulation / live。
- provider fetch、lake write、catalog publish。
- 自动启动 CR089 或恢复 CR020 用户删除的 gateway 路线。

请回复以下三者之一：

- `approve`
- `修改: <具体修改点>`
- `reject`
