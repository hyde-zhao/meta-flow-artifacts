# CP8 CR152 Human Gate Launch Message

请审查 `process/checkpoints/CP8-CR152-DELIVERY-READINESS.md`。

## 自动预检结论

本轮待人工决策项：1

| 项 | 结论 |
|---|---|
| CP8 result | PASS |
| release_decision | `READY_WITH_RISK` recommended |
| 阻断项 | 0 |
| 本轮待人工决策项 | 1 |
| 待人工决策项数量 | 1 |
| Context Capsule | `process/release/RELEASE-CONTEXT-CR152.yaml` |

## 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 完成 CR152 Machine Learning Strategy E2E first-wave framework 的 CP8 release readiness，决定是否按 static-fixture-only 范围以 `READY_WITH_RISK` 收尾。 |
| 推荐动作 | `approve`：接受 `DEC-CR152-CP8-001`，将 CR152 CP8 release decision 标记为 `READY_WITH_RISK`。 |
| approve 后会发生什么 | 记录用户接受 taxonomy/provenance hygiene 作为 CR152 scope-out follow-up candidate；CR152 local/static/fixture ML strategy E2E first-wave 按 `READY_WITH_RISK` 收尾；后续可选择 hygiene follow-up 或进入 CR153 rule-41 precheck。 |
| approve 不授权什么 | 不授权真实训练、真实数据验证、model registry 发布/写入、store/catalog 写入、runtime、lake/NAS/provider/QMT/broker/credential/external framework/Git remote，不授权测试 taxonomy/provenance cleanup，不表示真实 release execution。 |
| 不确认会阻塞什么 | 阻塞 CR152 CP8 closure；若用户不接受风险，则先处理 taxonomy/provenance hygiene 或将 CR152 标记 `NOT_READY`，再重新进入 CP8。 |

## 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 1 | `DEC-CR152-CP8-001` |
| 高风险策略确认 | 1 | release wording / non-authorized boundary must remain static-fixture-only and no real readiness claims. |
| agent 默认处理 | 4 | minimal profile、return-check warning non-blocking classification、CR151 process caveat carry-forward、rule 40 follow-up split formatting。 |
| 仅审计记录 | 1 | No true release execution; `RELEASED` / `FAILED` not used. |

## 决策收集覆盖

| 来源 | 纳入待决策数 | 说明 |
|---|---:|---|
| CP7 result/evidence/return | 1 | `CR152-CP7-R01` 进入 `DEC-CR152-CP8-001`；return-check warning 为 non-blocking。 |
| follow-up tracking | 1 | `FU-CR152-001` 必须作为 candidate 呈现，不自动升级正式 CR。 |
| release context | 1 | release decision、non-authorized items、forbidden release claims represented. |
| cr-tracking / workspace git-status | 0 | active formal CRs = CR-152; blocked formal CRs = none; no push/release execution. |

## 本轮待人工决策项

| 决策 ID | 决策类型 | 推荐方案 | 影响 |
|---|---|---|---|
| DEC-CR152-CP8-001 | risk_acceptance | 接受 taxonomy/provenance hygiene 作为 follow-up candidate，CR152 CP8 标记为 `READY_WITH_RISK`。 | 不声明 full pytest；不授权 cleanup；不影响 CR152 static-fixture-only 验证结论。 |

## 不授权范围

如果你回复 approve，不表示授权以下任何事项：

| 不授权项 | 状态 |
|---|---|
| 真实训练 / 真实数据验证 | not authorized |
| model registry 发布/写入、store/catalog 写入 | not authorized |
| runtime、lake/NAS/provider/QMT/broker/credential/external framework/Git remote | not authorized |
| 真实 release execution | not authorized |
| test taxonomy / provenance cleanup | not authorized |

## Exact Replies

请直接回复以下任一整行：

```text
approve
修改: <具体修改点>
reject
```
