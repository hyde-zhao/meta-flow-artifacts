# CP3 CR099 Human Gate Launch Message

请审查：`process/checkpoints/CP3-CR099-RUNNER-REAL-READONLY-SMOKE-HLD-REVIEW.md`

自动预检结论：PASS

Context Capsule：`process/context/CP3-CR099-RUNNER-REAL-READONLY-SMOKE-CONTEXT.yaml`

决策收集覆盖 / Decision Collection Coverage：已扫描 CP2 checkpoint、CR099 HLD、CP3 Architecture Gray Areas discussion、CP3 自动预检和 runtime authorization 边界；候选问题 16 项，纳入待人工决策 5 项，runtime 授权保持为不授权项。

本轮待人工决策项：5

如果你回复 approve，表示你接受以下 5 项推荐方案；不表示授权后面的禁止操作。

| 决策 ID | 决策类型 | 问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP3-CR099-01 | architecture | 是否接受 CR099 推荐架构？ | CR098 runner adapter + CR097 gateway pattern + CR099 run contract / `authorization_ref` / evidence schema | 用户手工 Windows-side 执行；或只做离线 reverify | 推荐方案最能证明 runner integration；备选更保守但覆盖较弱 | 影响 CP5 run contract 和 CP6/CP7 验证对象 |
| DQ-CP3-CR099-02 | runtime_authorization | CP3 approve 是否授权真实 run？ | 不授权；真实 run 必须 CP5 后逐 run 授权 | CP3 后立即运行；或永久禁止 runtime | 推荐方案保持设计批准与运行授权分离 | 当前只写设计，不触碰 secret/env/gateway/account |
| DQ-CP3-CR099-03 | security | env / HMAC / evidence 如何控制？ | run contract 只保存授权引用；evidence 只保存 redacted summary，forbidden counters 为 0 | 保存完整 CLI 输出；或隐式读取默认 env | 推荐方案最小化泄露面 | 影响 CP5 schema 和 redaction gate |
| DQ-CP3-CR099-04 | implementation | CP5 应固化什么？ | run manifest、evidence schema、preflight checklist、blocked-path checks 和测试计划；不执行 runtime | 直接执行 runner smoke；或只写文档 | 推荐方案可验证且不越权 | 影响 CP5 / CP6 / CP7 工作量 |
| DQ-CP3-CR099-05 | follow_up_tracking | 非空 / 交易日 / NAS / order-write 是否并入？ | 不并入 CR099，保持独立后续候选 | 合并非空 / 交易日；或合并 NAS / order-write | 推荐方案保持目标单一 | 合并会扩大权限和等待条件 |

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
