请审查：`process/checkpoints/CP8-CR156-HYGIENE-PACKAGING-TEST-TAXONOMY-RELEASE-READINESS.md`

自动预检结论：PASS，无 blocker。

Context Capsule：`process/context/CP8-CR156-HYGIENE-PACKAGING-TEST-TAXONOMY-CLOSURE-CONTEXT.yaml`，read_profile=compact，默认读取策略为 capsule-first。

审批者摘要：本次确认服务的整体目标是确认 CR156 是否可用现有 evidence 关闭两个 hygiene follow-up，并把 CR156 结束为 READY_WITH_RISK。推荐动作是 approve。approve 后会发生什么：CR156 标记为 closed / READY_WITH_RISK，`FU-CR154-001` 与 `FU-CR152-001` 标记为 closed under CR156，并清空当前 active CR / pending gate。approve 不授权什么：不授权 Git remote write / push、true release execution、publish、runtime、真实数据 lake/NAS/provider、凭据读取、broker、trading、catalog/store/registry 写入或外部框架运行。不确认会阻塞什么：CR156 继续停留在 active-cp8-pending-review。

决策分层：必须用户决策 1；高风险策略确认 1；agent 默认处理 3；仅审计记录 5。

决策收集覆盖：已扫描 CP2 review feedback、CP8 context capsule、closure summary；候选问题 3 项，纳入待决策 1 项；产品 / 设计 / Story 文档 N/A，原因是本 CR 不改变产品基线、设计或 Story。

本轮待人工决策项：1

| 决策 ID | 决策类型 | 推荐方案 | 备选方案 | 优劣 | 影响 / 风险 |
|---|---|---|---|---|---|
| DEC-CR156-CP8-001 | risk_acceptance | 批准 CR156 READY_WITH_RISK closure，关闭 `FU-CR154-001` 与 `FU-CR152-001`。 | A. 修改 closure wording 后再批准；B. reject 并保持 CR156 active。 | 推荐方案最贴合评审；A 可微调措辞但延迟关闭；B 保守但保留台账噪声。 | 不声明 full-suite green，不授权真实 release/runtime/external write。 |

完整表见 `process/checkpoints/CP8-CR156-HYGIENE-PACKAGING-TEST-TAXONOMY-RELEASE-READINESS.md`。

不授权项：Git remote write / push、true release execution、publish、runtime、真实数据 lake/NAS/provider、凭据读取、broker、trading、catalog/store/registry 写入、外部框架运行。

如果你回复 approve，表示接受上述 READY_WITH_RISK closure；不表示授权任何不授权项。

可回复：

- `approve`
- `修改: <具体修改点>`
- `reject`
