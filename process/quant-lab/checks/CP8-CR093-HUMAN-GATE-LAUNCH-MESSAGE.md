请审查：process/checkpoints/CP8-CR093-DELIVERY-READINESS.md

自动预检结论：PASS_WITH_RISK

Context Capsule / Release Context：process/release/RELEASE-CONTEXT-CR093.yaml（read_profile=compact，release_artifact_profile=compact，release_decision=READY_WITH_RISK）

本轮待人工决策项：5

决策收集覆盖：已扫描 CP7、Release Context、release docs 和 STATE；发现候选问题 6 个，纳入待决策 5 个。

如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权以下禁止操作。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP8-CR093-01 | risk_acceptance | 是否接受 CR093 READY_WITH_RISK 并关闭当前交付？ | 接受并关闭当前 ledger hygiene 交付。 | 修改后重提；reject 回 CP7/CP6。 | DoD 已达成，阻断项 0。 | CR093 可关闭，风险带入 follow-up。 |
| DQ-CP8-CR093-02 | risk_acceptance | 是否接受 R-CR093-01 warning-only backlog 保留？ | 接受 warning-only 不阻断。 | 本轮清零所有 warnings；开启 strict-warnings。 | 控制范围。 | 后续仍会看到 warning，但 exit 0。 |
| DQ-CP8-CR093-03 | risk_acceptance | 是否接受 R-CR093-02 standalone checker warning 明细差异？ | 接受，以主 CLI 为验收入口。 | 本轮收敛两套 checker 输出。 | 聚焦目标。 | standalone warning 细节少。 |
| DQ-CP8-CR093-04 | follow_up_tracking | CR093-FU-01/02 是否只保留候选？ | 保留 candidate-not-started。 | 本轮直接启动后续候选。 | 避免范围膨胀。 | 关闭后仍可追踪。 |
| DQ-CP8-CR093-05 | security | CP8 approve 是否授权 runtime、NAS、凭据、交易或 publish？ | 不授权。 | 另起独立 gate。 | 保持安全边界。 | 无外部副作用。 |

不授权项：

- QMT / MiniQMT / XtQuant / gateway / runner runtime。
- NAS、凭据、账户、资金、持仓、委托、成交或日志原文读取。
- submit / cancel、buy / sell、simulation / live。
- provider fetch、lake write、catalog publish。
- 真实 release execution / publish。
- 自动启动 CR093-FU-01、CR093-FU-02、CR091-FU-02、CR091-FU-03 或 CR020 gateway route restore。

请回复以下三者之一：

- `approve`
- `修改: <具体修改点>`
- `reject`
