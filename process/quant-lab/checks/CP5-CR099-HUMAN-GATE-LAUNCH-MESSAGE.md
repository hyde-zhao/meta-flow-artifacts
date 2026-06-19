# CP5 CR099 Human Gate Launch Message

请审查：`process/checkpoints/CP5-CR099-RUNNER-REAL-READONLY-SMOKE-READINESS.md`

自动预检结论：PASS

Context Capsule：`process/context/CP5-CR099-RUNNER-REAL-READONLY-SMOKE-CONTEXT.yaml`

决策收集覆盖 / Decision Collection Coverage：已扫描 CP3 checkpoint、CR099 HLD、Feature Design、LLD、TEST-PLAN 和 STATE pending queue；候选问题 33 项，纳入待人工决策 5 项，runtime 授权保持为不授权项。

本轮待人工决策项：5

如果你回复 approve，表示你接受以下 5 项推荐方案；不表示授权后面的禁止操作。

| 决策 ID | 决策类型 | 问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP5-CR099-01 | implementation | 是否批准当前 Feature Design / TEST-PLAN / TASKS / full LLD？ | 批准，进入 CP6 offline contract / schema checker implementation | 修改设计后重提；或 reject 回 CP3 approved | 推荐方案可执行且不越权；修改会延后；reject 暂停真实 smoke | 影响 run contract、evidence schema、tests 和后续 runtime 授权准备 |
| DQ-CP5-CR099-02 | architecture | run contract 是否作为真实 smoke 唯一入口？ | 是；真实 run 只能从 CP5 approved run contract + 用户逐 run 授权进入 | 用户手工 Windows-side 执行；或只做离线 reverify | 推荐方案可审计；备选更保守但覆盖较弱 | 影响 CP6 / CP7 证据形态 |
| DQ-CP5-CR099-03 | security | env / HMAC / evidence 是否按当前 schema 控制？ | contract 只保存 env ref，不保存 secret；evidence 只保存 redacted summary | 保存完整 CLI 输出；或隐式读取默认 env | 推荐方案最小泄露面；备选有 secret / 账户原文风险 | 安全风险 HIGH |
| DQ-CP5-CR099-04 | runtime_authorization | CP5 approve 是否授权真实 runner smoke？ | 不授权；只授权 CP6 离线 contract / schema implementation | 同时授权一次真实 runner smoke；或永久禁止 runtime | 推荐方案符合门禁；直接授权会越过逐 run 授权 | 防止 CP5 被误读为运行许可 |
| DQ-CP5-CR099-05 | follow_up_tracking | 非空 / 交易日 / NAS / order-write 是否独立？ | 独立；CR099 当前只做 runner real readonly smoke gate | 合并非空复测；合并 NAS；合并 order-write | 推荐方案保持单一目标；合并会扩大权限和风险 | 可能保留空持仓 / 非交易日剩余风险 |

不授权项：

- 不授权读取 HMAC secret、`/home/hyde/.quant-lab/runtime/qmt/cr097/client.env.template` 或 Windows `.env`。
- 不授权启动 Windows gateway、执行 runner runtime 或查询账户。
- 不授权读取 QMT 账号原文、资金、持仓明细原文、证券代码、委托、成交或原始日志。
- 不授权 NAS package exchange / pull / publish。
- 不授权 submit / cancel、buy / sell、simulation / live。
- 不授权 provider fetch、lake write、catalog publish。
- 不授权真实 release execution / publish。
- 不自动启动 CR089 或恢复 CR020 gateway route。

请回复以下三种之一：

- `approve`
- `修改: <具体修改点>`
- `reject`
