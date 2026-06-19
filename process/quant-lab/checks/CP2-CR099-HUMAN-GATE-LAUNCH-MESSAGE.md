# CP2 CR099 Human Gate Launch Message

请审查：`process/checkpoints/CP2-CR099-RUNNER-REAL-READONLY-SMOKE-SCOPE-REVIEW.md`

自动预检结论：PASS，阻断项 0。预检文件：`process/checks/CP2-CR099-RUNNER-REAL-READONLY-SMOKE-SCOPE-PRECHECK.md`

Context Capsule：`process/context/CP2-CR099-RUNNER-REAL-READONLY-SMOKE-CONTEXT.yaml`（read_profile=minimal；完整来源见 checklist）

决策收集覆盖 / Decision Collection Coverage：已扫描 6 个来源，发现候选问题 15 个，纳入待决策 5 个；N/A 来源 1 个，原因见 checklist 的 Decision Collection Coverage。

本轮待人工决策项：5

如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权以下 8 项禁止操作。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP2-CR099-01 | scope | 本轮 CR099 是否只覆盖 runner real readonly smoke？ | 仅覆盖 runner 调用 Windows gateway health / capabilities / query_positions_readonly 和 redacted evidence。 | 只做离线 reverify；合并非空 / 交易日复测。 | 推荐可关闭 CR098 真实 runner runtime 风险；离线 reverify 覆盖不足；合并复测会等待运行条件。 | 进入 CP3/CP5 和后续逐 run 授权。 |
| DQ-CP2-CR099-02 | runtime_authorization | CP2 approve 是否授权读取 HMAC env、启动 gateway 或执行 runner？ | 不授权；CP2 只授权进入设计门禁，真实 run 必须在 CP5 后逐 run 授权。 | 立即授权一次 runner smoke；永久禁止 runtime。 | 推荐避免设计批准被误读成运行授权；立即授权会越过 CP3/CP5。 | 当前不触碰 secret / env / account。 |
| DQ-CP2-CR099-03 | security | evidence 和敏感文件如何处理？ | 只允许 `.quant-lab/evidence/qmt/cr099/redacted/` 脱敏输出；禁止 raw payload、账号、证券代码、数量、资金、委托、成交和日志原文。 | 允许保存完整 CLI 输出；允许读取 Windows `.env` 自动配置。 | 推荐最小化敏感面；备选会带来 secret / 账户泄露风险。 | 需要 redaction gate 和 forbidden counters。 |
| DQ-CP2-CR099-04 | architecture | 后续 CP3/CP5 应采用哪条执行路径？ | 使用 CR098 runner adapter + CR097 gateway 模式，新增 CR099 run contract / authorization_ref / evidence schema。 | 用户手工执行后回传 evidence；新写独立脚本绕过 runner。 | 推荐证明 runner integration；手工回传可作 fallback；绕过 runner 不能关闭目标风险。 | 影响 CP3 HLD、CP5 run contract 和 CP6/CP7 证据。 |
| DQ-CP2-CR099-05 | follow_up_tracking | NAS、order-write、非空持仓复测如何处理？ | 全部保持独立候选，不并入 CR099。 | 合并 NAS；合并 order-write。 | 推荐保持单一目标；合并会扩大权限和风险。 | CR099 不解决包交换、下单或更广账户覆盖。 |

不授权项：
- 不读取 HMAC secret。
- 不读取 `/home/hyde/.quant-lab/runtime/qmt/cr097/client.env.template` 或 Windows `.env`。
- 不启动 Windows gateway。
- 不执行 runner runtime 或查询账户。
- 不读取 QMT 账号原文、资金、持仓明细原文、证券代码、委托、成交或原始日志。
- 不访问 NAS package exchange / pull / publish。
- 不执行 submit / cancel、buy / sell、simulation / live。
- 不执行 provider fetch、lake write、catalog publish、真实 release execution / publish，不自动启动 CR089 或恢复 CR020 gateway route。

回复 `approve` 表示接受上表全部推荐方案；如需调整，请用 `修改: <具体修改点>` 指明决策 ID 和修改内容。

审查后请直接回复以下任一整行：

approve
修改: <具体修改点>
reject
